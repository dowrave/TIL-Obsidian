- 타입을 파라미터로 사용할 수 있게 해주는 C#의 기능

## 클래스 정의 및 사용
```cs
public class GenericClass<T>
{
    private T item;

    public void SetItem(T newItem)
    {
        item = newItem;
    }

    public T GetItem()
    {
        return item;
    }
}

GenericClass<int> intClass = new GenericClass<int>();
intClass.SetItem(10);
int value = intClass.GetItem(); // value는 10

GenericClass<string> stringClass = new GenericClass<string>();
stringClass.SetItem("Hello");
string text = stringClass.GetItem(); // text는 "Hello"
```

## 메서드 정의 및 사용
```cs
public class Utilities
{
    public static void Swap<T>(ref T a, ref T b)
    {
        T temp = a;
        a = b;
        b = temp;
    }
}

int x = 5, y = 10;
Utilities.Swap(ref x, ref y);
```

## 제네릭에 제약 걸기
```cs
public class GenericWithConstraint<T> where T : IComparable
{
    public T CompareAndReturn(T a, T b)
    {
        if (a.CompareTo(b) > 0)
            return a;
        else
            return b;
    }
}
```
> 특정 인터페이스르 구현한 타입만 T로 사용 가능


