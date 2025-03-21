**PROSES PENGERJAAN**

1. Tombol pada layar game over untuk kembali ke menu utama.

    Untuk implementasi fitur ini saya menambahkan `LinkButton` di scene `Game Over` dan mengattach ke script `GameOverBtn` lalu menambahkan signal `pressed()` sehingga akan mengganti scene dengan scene `mainmenu` saat button dipencet serta melakukan beberapa layouting.

    Saya membuat script baru untuk button di `Game Over` karena terdapat logika tambahan yaitu mereset `life` sehingga kodenya menjadi seperti berikut
    ```
    extends LinkButton

    @export var scene_to_load: String

    func _on_Back_to_Menu_pressed():
        Global.lives = 3
        get_tree().change_scene_to_file(str("res://scenes/" + scene_to_load + ".tscn"))
    ```

2. Fitur select stage
    Untuk implementasi fitur ini pertama-tama saya membuat scene baru yaitu `Select Level.tscn` yang berisi satu `label` dan dua `LinkButton` untuk  memilih level 1 atau level 2 serta beberapa `container` untuk layouting. Selanjutnya saya  mengattach script `link_button` ke button `Stage Select` di `mainmenu` untuk signal pindah scene

    Untuk kedua button di `Select Level` saya mengattach script `link_button` untuk pindah ke scene level 1 atau level 2 dan juga menambahkan signal `pressed()`.

3. Fitur transisi 

    Untuk fitur ini saya membuat sebuah transisi sederhana yang akan muncul ketika `player` masuk ke `Area Trigger`. Untuk implementasi fitur ini pertama-tama saya membuat scene baru yaitu `transition_screen`, scene ini berisi dua node yaitu `ColorRect` dan `AnimationPlayer`. Selanjutnya saya mengubah properti `ColorRect` dan membuat dua animation yaitu `fade_to_black` dan `fade_to_normal`.

    Selanjutnya saya membuat script baru yaitu `transition_screen.gd` yang berisi kode berikut
    ```
    extends CanvasLayer

    signal on_transition_finished

    @onready var color_rect = $ColorRect
    @onready var animation_player = $AnimationPlayer

    func _ready():
        color_rect.visible = false
        animation_player.animation_finished.connect(_on_animation_finished)
        
    func _on_animation_finished(anim_name):
        if anim_name == "fade_to_black":
            on_transition_finished.emit()
            animation_player.play("fade_to_normal")
        elif anim_name == "fade_to_normal":
            color_rect.visible = false
        
    func transition():
        color_rect.visible = true
        animation_player.play("fade_to_black")
    ```
    Selanjutnya saya menambahkan scene `transition_screen` sebagai global variable dan memodifikasi kode `Area2D` menjadi seperti berikut
    ```
    extends Area2D

    @export var sceneName: String = "Level 1"

    func _on_Area_Trigger_body_entered(body):
        var current_scene = get_tree().get_current_scene().get_name()
        if body.get_name() == "Player":
            if current_scene == sceneName:
                Global.lives -= 1
            if (Global.lives == 0):
                get_tree().change_scene_to_file(str("res://scenes/Game Over.tscn"))
            else:
                TransitionScreen.transition()
                await TransitionScreen.on_transition_finished
                get_tree().call_deferred("change_scene_to_file",(str("res://scenes/" + sceneName + ".tscn")))
    ```

4. Fitur back to menu

    Fitur ini saya buat sehingga `player` dapat kembali ke main menu saat sudah masuk ke stage level. Untuk membuat fitur ini saya membuat scene baru yaitu `Back To Menu` dan menambahkan `MarginContainer` dan `LinkButton` di scene tersebut. Selanjutnya saya mengattach script `GameOverBtn.gd` ke `LinkButton` karena memiliki logika yang sama dan menambahkan signal `pressed()`. Terakhir saya menambahkan scene `Back To Menu` ke scene `Level 1` dan `Level 2`

5. Polishing lainnya
    
    Untuk proses polishing lain yang saya lakukan adalah mengganti text button di main menu menjadi "New Game" dan menambahkan life counter pada scene level 2

<br>
Referensi transisi scene:

- https://www.youtube.com/watch?v=Shj_QVwrefY