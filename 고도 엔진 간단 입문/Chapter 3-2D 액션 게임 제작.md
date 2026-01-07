# Chapter 3. 2D 액션 게임 제작

```GDScript
extends CharacterBody2D

@export var WALK_SPEED = 250
@onready var sprite = $AnimatedSprite2D

@onready var direction = Vector2(0,0)

@export var GRAVITY = 15
@export var JUMP_POWER = 450

@onready var raycast2d = $RayCast2D

func _ready():
	self.z_index = 100
	
func _physics_process(delta: float) -> void:
	if Input.is_action_pressed("ui_right"):
		direction.x = WALK_SPEED
		sprite.flip_h = false
		sprite.play("walk")
	elif  Input.is_action_pressed("ui_left")	:
		direction.x = -WALK_SPEED
		sprite.flip_h=true
		sprite.play("walk")
	else:
		direction.x=0
		sprite.play("idle")	
	
	direction.y += GRAVITY
	if raycast2d.is_colliding():
		direction.y = 0
		if Input.is_action_just_pressed("ui_up"):
			direction.y = -JUMP_POWER
			
	self.velocity = direction
	self.move_and_slide()
```

- @export : 변수를 인스펙터에서 변경할 수 있도록 한다
- @onready : 이 스크립트가 설정되어 있는 노드와 그 자식 노드의 초기화가 완료됐을 때 실행한다.

```GDScript
extends Area2D

@export var HP_RECOVERY = 10

# Called when the node enters the scene tree for the first time.
func _ready() -> void:
	self.z_index=1


# Called every frame. 'delta' is the elapsed time since the previous frame.
func player_hp_up(body):
	if body.name = "Player":
		body.hp_up(HP_RECOVERY)
		self.queue_free()
```

- self.queue_free() : 자기 자신 요소 삭제