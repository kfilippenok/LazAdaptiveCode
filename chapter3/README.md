# Глава 3. Зависимости и разделение на уровни.

## Зависимости

Зависимость - это отношение между двумя сущностями. Оно состоит из *Клиента* и *Службы*. Клиент всегда зависит от службы.

```mermaid
flowchart LR
    A["`(A)
	Клиент`"] -->|Зависит от| B["`(B)
	Служба`"]
```

### Зависимость от инфраструктуры

В качестве инфраструктуры у нас выступают *Lazarus* и компилятор *FreePascal*, через который *Lazarus* компилирует наш проект.

### Сторонние зависимости

К сторонним зависимостям относятся сборки других разработчиков. Ими могут быть пакеты в *Lazarus*, или модули, добавленные в проект.

Пример сторонней зависимости в виде пакета *BGRABitmapPack*:

![](media/dependencie_bgrabitmappack.png) 

### Моделирование зависимостей

#### Моделирование зависимостей в ориентированном графе

Для моделирования зависимостей лучше всего подходят *ориентированные графы*.

Пример *ориентированного графа*:
```mermaid
flowchart TD
	A((A)) --> B((B))
	A((A)) --> C((C))
	B((B)) --> E((E))
	B((B)) --> D((D))
	B((B)) --> F((F))
	D((D)) --> F((F))
```

#### Циклические зависимости

Предыдущий граф является *ациклическим*, т.к. он не имеет циклов. Но существуют так же и *цилкические* графы, в которых можно начать и вернуться в один и тот же узел.

Пример *циклического орграфа*:
```mermaid
graph
	A((A)) --> B((B))
	A((A)) --> C((C))
	B((B)) --> D((D))
	E((E)) --> B((B))
	D((D)) --> E((E))
```

Так делать **нельзя**. Это называется *циклической зависимостью*, иногда называемой *круговой заисимостью*. В таком случае узел ``D`` зависит *сам от себя*, чего быть **не должно**.

Для примера возьмём два модуля, которые буду зависить друг от друга.

Модуль A:

```Pascal
unit A;

{$mode ObjFPC}{$H+}

interface

uses
  Classes, SysUtils, B;

implementation

end.    
```

Модуль B:

```Pascal
unit B;

{$mode ObjFPC}{$H+}

interface

uses
  Classes, SysUtils, A;

implementation

end.  
```

Зависимость между ними выглядит вот так:

```mermaid
graph
	A((A)) --> B((B))
	B((B)) --> A((A))
```

Компилятор Free Pascal не даст вам сделать циклическую зависимость, выдав ошибку:

```
a.pas(8,23) Error: Circular unit reference between A and B
```

Однако есть *уловка*, позволяющая модулям использовать друг друга. Для этого достаточно в одном из модулей переместить второй модуль в ``uses`` из раздела ``interface`` в раздел ``implementation``:

Модуль А:

```Pascal
unit A;

{$mode ObjFPC}{$H+}

interface

uses
  Classes, SysUtils;

implementation

uses
  B;

end.        
```

Модуль ``B`` осталься таким же. Но это **плохой пример**, который использовать не стоит.

#### Петли

*Петли* являются специализациями циклов в орграфах. Если узел соединяется с помощью ребра с *самим собой*, тогда ребро становится петлёй.

На практике сборки всегда явно зависят от себя. и такое наблюдение не заслуживает особого внимания. Однако на уровне методов и функций петля свидетельствует о *рекурсии*:

```Pascal
interface

type

  TRecursionLoop = class
  public
    procedure A;
    function B(ANumber: Integer): Integer;
  end;
  
implementation

  procedure TRecursionLoop.A;
  var 
    x: Integer = 6;
  begin
    WriteLn(Format('%d <> %d', [x, B(x)]));
  end;
  
  function TRecursionLoop.B(ANumber: Integer): Integer;
  begin
    if ANumber = 0 then 
      Exit(1)
    else
      Exit(ANumber * B(ANumber - 1));
  end;
```

Их зависимость выглядит так:

```mermaid
graph
	A((A)) --> B((B))
	B((B)) --> B((B))
```

#### Просмотр зависимостей

В *Lazarus* используется *View -> Unit Dependencies*.

![](media/unitdependencies.png)

## Управление зависимостями

### Реализация или интерфейсы

Клиенту интерфейса не нужно знать как он реализован, т.к. это привязывает к конкретной реализации.

Приведение ссылки на интерфйес к любой релизации - **всегда** плохая идея.

### Ключевое слово new  как признак плохого кода

Для Object Pascal, с учётом контекста последующего текста, это можно сказать так: "Создание экземпляров внутри класса как признак плохого кода".

**Листинг 3.6 Пример того, как создание экземпляров лишает код возможности быть адаптивным**:

```Pascal
TAccountController = class
strict private
  FSecurityService: TSecurityService;
public
  constructor Create; override;
  procedure ChangePassword(AUserID: TGUID; ANewPassword: String);
end;

constructor TAccountController.Create;
begin
  FSecurityService := TSecurityService.Create;
end;

procedure TAccountController.ChangePassword(AUserID: TGUID; ANewPassword: String);
var
  UserRepository: TUserRepository;
  User: TUser;
begin
  UserRepository := TUserRepository.Create;
  User := UserRepository.GetByID(AUserID);
  Self.FSecurityService.ChangePassword(User, ANewPassword) 
end;
```

Данный пример в оригинале имеет контекст с ASP.NET MVC, в рамках которого и существует контроллер. Нам важно понимать, что обязанность контроллера заключается в том, чтобы позволить пользователю выполнять запросы и команды, связанные с учётной записью. 

## Разделение на уровни


## Заключение