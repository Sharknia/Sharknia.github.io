---
IDX: "NUM-116"
tags:
  - Work
  - FastAPI
  - SqlAlchemy
  - Python
description: "Sqlalchemy 2.0 비동기 엔진과 페이지네이션 라이브러리 직접 구현"
update: "2024-02-06T06:34:00.000Z"
date: "2024-01-17"
상태: "Ready"
title: "FastAPI의 Pagenation"
---
## 서론

FatAPI에는 페이징을 위한 [공식 라이브러리](https://uriyyo-fastapi-pagination.netlify.app/)가 존재한다. 하지만 예제대로 진행해도 코드는 오류를 내뿜었다. 왜냐하면, FastAPI의 페이지네이션 라이브러리는 SqlAlchemy 2.0의 비동기 엔진을 지원하지 않기 때문이다. 

그래서 직접 구현했다. 

~~(이~~ [~~라이브러리~~](https://pypi.org/project/fastapi-sqla/)~~를 쓰면 Async pagination 지원합니다. 여러분은 이거 쓰세요.)~~

## 목적

### 응답값 고정

모든 페이징 쿼리에 대해 동일한 응답값을 제공하기 위해 Response에 사용할 Pydantic Model도 함께 정의한다. 

### 가능한 한 간단한 사용 방법

Sqlalchemy의 Select 객체를 받아 바로 처리할 수 있도록 한다. 즉, 개발자는 쿼리문만 생성하고 사이즈/페이지만 정해주면 바로 고정된 응답값을 받을 수 있다. 

### 현재 서버에 필요한 사양에 맞춘 기능

복잡한 쿼리나 여러 층의 정렬을 현재는 필요로 하지 않는다. 이에 발맞춰 기본적인 relationship을 활용한 조인, 단일 컬럼 sort만 지원하여 사용법을 간략화 한다. 

## 구현

### PaginationReturn Class 구현

페이지네이션의 결과의 데이터 모델을 미리 정의해 둔 클래스이다. 

응답값은 다음의 내용들로 고정된다.

- itemList : 페이지에 포함된 Row의 리스트

- totalCount : 전체 항목의 개수

- totalPage : 전체 페이지의 개수 

- nowPage : 현재 페이지 번호

- nowSize : 현재 설정한 사이즈 

- prevPage : 이전 페이지 번호

- nextPage : 다음 페이지 번호

<details>
<summary>전체 코드</summary>

```yaml
class PaginationReturn(BaseModel, Generic[T]):
    """
    PaginationReturn 클래스는 페이지네이션 결과를 나타내는 모델입니다.

    Attributes:
        itemList (List[T]): 페이지에 포함된 항목의 리스트입니다.
        totalCount (int): 전체 항목의 개수입니다.
        totalPages (int): 전체 페이지의 개수입니다.
        nowPage (int): 현재 페이지 번호입니다.
        nowSize (int): 현재 페이지에 포함된 항목의 개수입니다.
        prevPage (Optional[int], optional): 이전 페이지 번호입니다. 기본값은 None입니다.
        nextPage (Optional[int], optional): 다음 페이지 번호입니다. 기본값은 None입니다.
    """
    itemList: List[T]
    totalCount: int
    totalPages: int
    nowPage: int
    nowSize: int
    prevPage: Optional[int] = None
    nextPage: Optional[int] = None
```


</details>

### Pagination Class 구현

pagination은 static 메소드로 만들어서 필요한 값을 넣으면 바로 PaginationReturn을 받을 수 있게 만들었다. static method로 만든것은 FastAPI의 Pagination 라이브러리를 본딴것이다. 이 방법이 가장 사용하기 편해보이기도 했다. 

#### `get_paginated_list`

처음 구현한 메소드이다. 모델을 파라미터로 받아 페이징을 한다. 따로 select문을 만들 필요 없이 모델과 where절만 만들어서 넣어줘도 정의된 응답값 데이터 모델에 리스트를 담아 반환해준다. 

<details>
<summary>전체 코드</summary>

```python
@classmethod
    async def get_paginated_list(
        cls,
        db: AsyncSession,
        model: T,
        filters: Optional[List[BinaryExpression]] = None,
        order_column: Optional[str] = None,
        order_direction: str = "desc",
        size: int = 10,
        page: int = 1,
    ) -> PaginationReturn:
        # page 와 size 기본 유효성 검증
        if page < 1 or size < 1:
            raise PagingException("Page and size parameters must be greater than 0")

        # 기본 쿼리 생성
        query = select(model)

        # fileter 처리
        if filters and len(filters) > 0:
            query = query.filter(*filters)

        # 정렬 처리
        if order_column:
            if order_direction.lower() == "asc":
                query = query.order_by(asc(getattr(model, order_column)))
            else:
                query = query.order_by(desc(getattr(model, order_column)))

        total_count = 0
        try:
            # 페이징
            offset_value = (page - 1) * size
            query = query.offset(offset_value).limit(size)
            result = await db.execute(query)
            items = result.scalars().all()
            logger.info(f"[Pagination Query] {query}")
            # total_count 계산
            total_query = select(func.count()).select_from(model)
            if filters:
                total_query = total_query.filter(*filters)
            total_result = await db.execute(total_query)
            total_count = total_result.scalar_one()
        except SQLAlchemyError as e:
            raise PagingException(f"An error occurred while fetching data from the database : {e}")
        except Exception as e:
            raise PagingException(f"An unexpected error occurred : {e}")

        total_pages, prev_page, next_page = calculate_pagination(total_count, size, page)

        # 응답값 생성
        items_dict = [serialize_sqlalchemy_obj(item) for item in items]

        res = PaginationReturn(
            itemList=items_dict,
            totalCount=total_count,
            totalPages=total_pages,
            nowPage=page,
            nowSize=size,
            prevPage=prev_page,
            nextPage=next_page,
        )
        return res
```


</details>

BaseModel 객체는 직렬화 기능이 없으므로, Response로 응답을 내려보낼때에 귀찮아지는 문제가 있다. 모델의 키값과 밸류를 사용해 dictionary로 바꾸면 해당 문제를 해결할 수 있는데, 만약 리스트에 들어있는 값들에 대해 커스텀이 필요하다면 dictionary를 그대로 다루는 것은 아무래도 모델을 커스텀 하는 것보다 불편한 문제가 발생한다. 

이 문제를 해결하기 위해 모델 객체를 기반으로 Pydantic Model 객체를 동적으로 생성하여 Pydantic model의 리스트를 결과값에 담아 리턴하기로 했다. 이렇게 하면 응답값을 바로 Response로 내려보내도 직렬화가 가능해 문제가 발생하지 않으며, 혹시 아이템 내용을 커스텀 해야 할 때에도 모델을 다룰 때와 같은 방법으로 사용할 수 있으므로 훨씬 간편하다. 개선된 코드는 아래와 같다. 

<details>
<summary>개선된 코드</summary>

```python
@classmethod
    async def get_paginated_list(
        cls,
        db: AsyncSession,
        model: T,
        filters: Optional[List[BinaryExpression]] = None,
        order_column: Optional[str] = None,
        order_direction: str = "desc",
        size: int = 10,
        page: int = 1,
    ) -> PaginationReturn:
        """
        페이징을 해서 아이템을 뽑아내서 결과를 반환한다.
        Args:
            db: AsyncSession
            model: 데이터를 가져올 모델
            filters: [InitQuizList.is_deleted == False, InitQuizList.is_active == True] 의 꼴 where절
            order_column: 정렬할 컬럼
            order_direction: 'asc' or 'desc'
            size: 페이지에 나타낼 아이템의 개수 (기본값 10)
            page: page 번호 (기본값 1)
        Returns:
            PaginationReturn
        """

        # page 와 size 기본 유효성 검증
        if page < 1 or size < 1:
            raise PagingException("Page and size parameters must be greater than 0")

        # 기본 쿼리 생성
        query = select(model)

        # fileter 처리
        if filters and len(filters) > 0:
            query = query.filter(*filters)

        # 정렬 처리
        if order_column:
            if order_direction.lower() == "asc":
                query = query.order_by(asc(getattr(model, order_column)))
            else:
                query = query.order_by(desc(getattr(model, order_column)))

        total_count = 0
        try:
            # 페이징
            offset_value = (page - 1) * size
            query = query.offset(offset_value).limit(size)
            result = await db.execute(query)
            items = result.scalars().all()
            logger.info(f"[Pagination Query] {query}")
            # total_count 계산
            total_query = select(func.count()).select_from(model)
            if filters:
                total_query = total_query.filter(*filters)
            total_result = await db.execute(total_query)
            total_count = total_result.scalar_one()
        except SQLAlchemyError as e:
            raise PagingException(f"An error occurred while fetching data from the database : {e}")
        except Exception as e:
            raise PagingException(f"An unexpected error occurred : {e}")

        total_pages, prev_page, next_page = calculate_pagination(total_count, size, page)

        # 응답값 생성
        # 동적 Pydantic 모델 생성
        PydanticModel = sqlalchemy_to_pydantic(model)
        # 페이징 처리된 쿼리 결과를 Pydantic 모델 리스트로 변환
        pydantic_items = [PydanticModel.model_validate(item.__dict__) for item in items]

        res = PaginationReturn(
            itemList=pydantic_items,
            totalCount=total_count,
            totalPages=total_pages,
            nowPage=page,
            nowSize=size,
            prevPage=prev_page,
            nextPage=next_page,
        )
        return res

def sqlalchemy_to_pydantic(db_model: Type[DeclarativeMeta]) -> Type[BaseModel]:
    """
    SQLAlchemy 모델을 동적으로 생성된 동일한 스키마의 Pydantic 모델로 변환합니다.
    :param db_model: SQLAlchemy 모델 클래스
    :return: 생성된 Pydantic 모델 클래스
    """
    fields = {}
    for column in inspect(db_model).c:
        python_type = column.type.python_type
        default = None if column.default is None else column.default.arg
        if column.nullable:
            python_type = Optional[python_type]
        fields[column.name] = (python_type, default)

    pydantic_model = create_model(db_model.__name__ + "Pydantic", **fields)
    return pydantic_model
```

모델의 컬럼값으로 동적으로 pydantic model을 생성하는 sqlalchemy_to_pydantic 메소드를 생성하고 해당 메소드를 활용해 응답값에 담도록 해주었다. 


</details>

#### `get_paginated_list_by_query`

처음에 신나서 만들었는데, 기존  `get_paginated_list` 메소드는 치명적인 문제가 있다. 쿼리가 복잡한 경우를 전혀 생각하지 않았다. 말 그대로 단일 테이블에서만 값을 가져올 수 있는 것이다. 그래서 이 점을 개선해야했다. 그래서 개발자가 조금 더 귀찮아지지만 쿼리문은 이 메소드를 이용할 작업자가 직접 구현을 하고, 그 쿼리문을 넣으면 리스트를 담아서 돌려주는 형식으로 다시 만들기로 했다. 

기존 메소드에서는 모델만을 조회하는 것이므로 모델을 기반으로 Pydantic model을 생성하면 문제가 없었는데, 이번에는 모델에서 컬럼값을 추출해서 사용할 수 없으므로 키 값을 동적으로 정의하는 코드를 따로 작성해주었다. 

쿼리된 내용에서 컬럼값을 조회해서 사용하였으며, 쿼리된 내용에는 시스템에서 사용하는 속성값도 존재하는데 해당 속성값들은 `_`로 시작하므로 해당 내용은 제거하고 동적으로 Pydantic model에 추가해주었다. relationship도 대응할 수 있게 별도의 예외처리를 추가해주었다. relationship 속성은 Select 객체의 컬럼에는 포함이 되지 않는데 조회된 결과물의 속성에는 들어있으므로 해당 값이 relationship 속성이라고 가정하고 따로 예외처리를 해주었다. 

<details>
<summary>전체 코드</summary>

```python
@classmethod
    async def get_paginated_list_by_query(
        cls,
        db: AsyncSession,
        query: Select[Any],
        size: int = 10,
        page: int = 1,
    ):
        """
        Query에 대한 paging 생성
        Args:
            db: AsyncSession
            query: sqlalchemy의 Select의 리턴값
            size: 한 번에 보여줄 아이템의 개수(기본값 10)
            page: 페이지(기본값 1)
        Returns:
            PaginationReturn
        """
        # 매개변수 검증
        if page < 1 or size < 1:
            raise PagingException("Page and size parameters must be greater than 0")

        # 페이징 적용
        offset_value = (page - 1) * size
        paginated_query = query.offset(offset_value).limit(size)

        # 쿼리 실행
        try:
            result = await db.execute(paginated_query)
        except SQLAlchemyError as e:
            raise PagingException(f"An error occurred while fetching data from the database : {e}")
        except Exception as e:
            raise PagingException(f"An unexpected error occurred : {e}")
        items = result.scalars().all()

        pydantic_items = []
        total_count = 0
        if items:
            # 컬럼과 데이터 타입 파악
            columns = query.columns
            fields = {col.name: (Optional[col.type.python_type], None) for col in columns}

            # relationship 처리
            relationship_keys = set()
            sample_item = items[0]
            item_dict = sample_item.__dict__
            for key in item_dict.keys():
                if key not in fields and not key.startswith("_"):
                    # 관계형 속성의 키를 저장
                    relationship_keys.add(key)

            # 관계형 속성에 대한 필드 정의 추가
            for key in relationship_keys:
                fields[key] = (Optional[Any], None)
            # 관계형 속성을 포함한 동적 Pydantic 모델 생성
            DynamicPydanticModel = create_model("DynamicPydanticModel", **fields)

            for item in items:
                item_dict = {k: v for k, v in item.__dict__.items() if not k.startswith("_")}
                # 관계형 속성을 딕셔너리로 변환
                for key in relationship_keys:
                    relation = getattr(item, key, None)
                    if relation:
                        # SQLAlchemy 내부 상태 정보를 제외하고 변환
                        item_dict[key] = {
                            k: v for k, v in relation.__dict__.items() if not k.startswith("_")
                        }
                    else:
                        item_dict[key] = None

                pydantic_items.append(DynamicPydanticModel.model_validate(item_dict))

            # 총 개수 계산
            total_count_query = select(func.count()).select_from(query.subquery())
            total_count_result = await db.execute(total_count_query)
            total_count = total_count_result.scalar_one()

        # 결과 반환
        total_pages, prev_page, next_page = calculate_pagination(total_count, size, page)

        return PaginationReturn(
            itemList=pydantic_items,
            totalCount=total_count,
            totalPages=total_pages,
            nowPage=page,
            nowSize=size,
            prevPage=prev_page,
            nextPage=next_page,
        )
```


</details>

## 결론

아직 기능이 많이 부족하지만, 현재 회사에서 사용을 하면서 필요한 기능은 대부분 구현을 했다고 생각이 든다. 그래서 일단 개발은 여기서 멈추었다. 

개발에 아주 오랜 시간이 걸리지는 않았다. 집중도와 실제 사용 시간을 따지면 이틀 좀 안되게 썼다고 생각이 든다. 

만드는것은 재미있는 과정이었고, 필요한 것이 잘 만들어져 뿌듯함도 있었지만 역시 있는 라이브러리를 쓰는게 시간 최적화에서는 더 좋았던 것 같다. 



