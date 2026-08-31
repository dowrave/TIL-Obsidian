- 이름이 없는 함수, 흔히 람다식을 일컬음

```cs
onConfirm: () => SaveSlotManager.Instance.DeleteSlot(index)
```

- `() => ...` 부분이 익명 함수로, "이름 붙은 메서드를 별도로 정의하지 않고 그 자리에서 즉석으로 만든 함수"라는 의미다.
- `Action`, `Fucn<T>` 등의 델리게이트 타입에 대입할 값을 만들 때 쓴다. 