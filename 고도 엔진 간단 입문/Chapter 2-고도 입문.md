# Chapter 2. 고도 입문

GDScript는 `동적 타입 언어` 지만 타입을 명시해서 변수를 지정하는 `정적 타이핑`도 가능

### 열거형

```GDScript
enum Days { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }
print(Days.MONDAY) # 0 출력
print(Days.FRIDAY) # 4 출력
```

### 배열

```GDScript
var arr: Array = [1, 2, 3, 4, 5]

arr.append(6)
print(arr) # [1, 2, 3, 4, 5, 6] 출력

arr.remove(2)
print(arr) # [1, 2, 4, 5, 6] 출력

arr.insert(2, 3)
print(arr) # [1, 2, 3, 4, 5, 6] 출력

arr.size() # 배열 크기 반환


```

### 딕셔너리

키에는 문자열만 사용 가능함. 배열과 달리 순서 개념이 없음

```GDScript
var dict: Dictionary = {
    "name": "Godot",
    "version": 4.0,
    "is_open_source": true
}

print(dict["name"]) # "Godot" 출력
dict["version"] = 4.1
print(dict) # {"name": "Godot", "version": 4.1, "is_open_source": true} 출력

dict.erase("is_open_source")
print(dict) # {"name": "Godot", "version": 4.1} 출력
```

### 연산

```GDScript
var a: int = 10
var b: int = 3

print(a + b) # 13
print(a - b) # 7
print(a * b) # 30
print(a / b) # 3.3333
print(a % b) # 1

print(a == b) # false
print(a != b) # true
print(a > b)  # true
print(a < b)  # false
print(a >= b) # true
print(a <= b) # false

var c: Array = [1, 2, 3]
var x: int = 2

print(x in c) # true    
print(x not in c) # false
```


### 처리 제어

```GDScript
var score: int = 85
if score >= 90:
    print("A 학점")
elif score >= 80:
    print("B 학점")
else:
    print("C 학점 이하")
```

```GDScript
for i in range(5):
    print(i) # 0부터 4까지 출력
```

```GDScript
var day: int = 3
match day:
    1:
        print("월요일")
    2:
        print("화요일")
    3:
        print("수요일")
    4:
        print("목요일")
    5:
        print("금요일")
    6:
        print("토요일")
    7:
        print("일요일")
    _:
        print("잘못된 요일")
```

### 함수

```GDScript
func add(a: int, b: int) -> int:
    return a + b

var result: int = add(5, 10)
print(result) # 15 출력
```

### 클래스 및 객체 지향 프로그래밍

```GDScript
class_name Person
var name: String
var age: int

func _init(name: String, age: int):
    self.name = name
    self.age = age

func greet() -> void:
    print("안녕하세요, 제 이름은 %s이고 나이는 %d살입니다." % [name, age])
```


## 주요 게임 요소
### 물리특성
- RigidBody3D: 물리 기반 객체, 게임 안의 장애물, 캐릭터의 공격
- StaticBody3D: 고정된 물리 객체. 지면, 계단, 벽 등
- KinematicBody3D: 스크립트로 제어되는 물리 객체
- CollisionShape3D: 충돌 영역 정의
- Area3D: 충돌 감지 및 트리거 영역
- CharacterBody3D: 캐릭터 제어용 물리 객체

### 애니메이션
- AnimationPlayer: 애니메이션 정보가 등록된 객체
- AnimationTree: AnimationPlayer에 등록되어 있는 애니메이션의 전환에 대한 제어
- AnimatedSprite2D, AnimatedSprite3D: 2D 스프라이트 애니메이션을 3D 공간에서 표현