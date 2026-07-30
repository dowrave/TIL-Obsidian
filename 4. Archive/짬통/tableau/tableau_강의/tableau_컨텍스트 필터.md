> 필터 선택에 따라 차원이 드릴다운되는 화면을 만들어보시오

1. 지역을 필터로 쓴다
2. 드릴다운을 위해, 모든 지역이 선택되었을 때와 일부 지역이 선택되었을 때가 나뉘어야 함
3. 전자의 경우 차원은 `Region`, 특정 지역은 `State`로 가져와야 함
4. 이 때 쓸 수 있는게 `IF` 문임.

```tableau
IF ALL 선택 THEN [REGION] ELSE [STATE] END

// OR

IF REGION 선택 THEN [STATE] ELSE [REGION] END
```
- 이라는 특성을 만들어야 함
- 주의할 것 ) 
1. `Region`, `State`는 집계되지 않은 `Raw Level`임!
	- IF문 내에 들어가는 값들은 통일돼야 함 : 모두가 집계값이든가, 모두가 단일값이든가.
2. **`ALL`이나 `특정 지역`이 선택되었을 때는 전체 레코드 수가 달라짐** 

--> 집계되지 않은 값을 쓰려면 `Fixed LOD`를 써야 함

```tableau
IF
{SUM(Number of Records)} =
{FIXED [Region] : SUM([Number of Records])}
THEN
	[State]
ELSE
	[Region]
END
```

테스트
> Quantity, State or Region(만든 특성)을 올린다  
> Region을 필터에 올린다
> Central만 활성화시킨다. 

- 특정 지역만 선택되었는데, 차원이 `주` 레벨로 나오지 않고 `Region` 레벨로 나오고 있는 걸 확인할 수 있다

#### 주 레벨로 드릴다운되지 않는 이유
- 위 필터가 제대로 바뀌기 위한 조건은, **2개의 Fixed LOD가 필터에 의해 값이 바뀌어야 한다.**
- 근데 **{Fixed}** 는 차원필터보다 먼저 적용됨 : 즉, 차원 필터가 적용되든 말든 구하는 값이 동일하다는 의미임
- 따라서 **차원 필터를 {Fixed}보다 먼저 적용되는 컨텍스트 필터로 만들어야 함**

- 적용법은 간단함 : `Region 필터 우클릭 - 컨텍스트에 추가`만 적용하면 됨
- 한 **Region**을 선택했을 때, `주` 이름들이 쫘르륵 나열되면 성공임(2개 이상은 Region으로 남아 있음)