
## 정의
>[!note]
>- GoF 정의
>- 서브시스템에 있는 일련의 인터페이스에 대한 통합된 인터페이스를 제공한다. `Facade`는 서브시스템을 더 쉽게 사용할 수 있게 해주는 상위 레벨 인터페이스를 제공한다.

## 핵심
1. 복잡한 서브 시스템들이 존재한다.
2. 그것들을 몰라도 되는 단순한 창구 하나를 제공한다.
3. 서브시스템은 여전히 직접 접근 가능하다.
	- 이 패턴은 "편의를 위한 선택지"를 제공하는 데 의의가 있다. **서브시스템을 꽁꽁 감추는 것 여부가 목적이 아님!**

## 예시
```cs
class ComputerFacade
{
    private CPU _cpu;
    private Memory _memory;
    private HardDrive _hardDrive;

    public void Start()
    {
        _cpu.Freeze();
        _memory.Load(BootAddress, _hardDrive.Read(BootSector, SectorSize));
        _cpu.Jump(BootAddress);
        _cpu.Execute();
    }
}
```
CPU, 메모리, 하드드라이브 모두 초기화 순서와 방법이 다르지만, `ComputerFacade.Start()` 하나로 내부에서 복잡한 순서를 다 정리해주는 방식이다.

### 포인트
- 캡슐화보다 편의성/단순화에 가깝다. 서브시스템 클래스들이 `public`이어도 상관 없다.
