---
title: Taichi 0.7.22 GUI 系统
date: 2021-07-24 04:00:00 +0800
categories: [CG, SIMULATION]
tags: [taichi]     # TAG names should always be lowercase
math: true
pin: false
---

## 创建窗体

1. `ti.GUI(title = 'Taichi', res = (512, 512), background_color = 0x000000, show_gui = True, fullscreen = False)`

* res：标量或元组。

* background_color：十六进制 rgb。

```python
gui = ti.GUI('Window Title', (640, 360))
```

2. `gui.show(filename = None)` 

指定文件名将保存截图。

## 在窗体上绘制

太极 GUI 支持绘制简单的几何对象。位置(0.0, 0.0)表示窗体左下角，位置(1.0, 1.0)表示窗体右上角。

1. `gui.set_image(img)`

图像像素由 img[i, j]，i 为横坐标，j 为纵坐标。如果窗体的大小为(x, y)，则 img 必须为以下之一：

* `ti.field(shape=(x, y))` 灰度图

* `ti.field(shape=(x, y, 3))` rgb

* `ti.field(shape=(x, y, 2))` rg

* `ti.Vector.field(3, shape=(x, y))` rgb

* `ti.Vector.field(2, shape=(x, y))` rg

* `np.ndarray(shape=(x, y))`

* `np.ndarray(shape=(x, y, 3))`

* `np.ndarray(shape=(x, y, 2))`

img 的数据类型必须为以下之一：

* `uint8` [0, 255]

* `uint16` [0, 65535]

* `uint32` [0, 4294967295]

* `float32` [0, 1]

* `float64` [0, 1]

2. `gui.get_image()`  

获取 rgba 图像，返回值类型为 np.array。

3. `gui.circle(pos, color = 0xFFFFFF, radius = 1)`

* pos：二维元组。

* color：十六进制 rgb。

4. `gui.circles(pos, color = 0xFFFFFF, radius = 1)`

* pos：np.array。

* color：十六进制 rgb 或 uint32 类型的 np.array。如果 color 是一个 numpy 数组，pos[i] 处的圆的颜色为 color[i]。

5. `gui.line(begin, end, color = 0xFFFFFF, radius = 1)`

6. `gui.lines(begin, end, color = 0xFFFFFF, radius = 1)`

7. `gui.triangle(a, b, c, color = 0xFFFFFF)`

8. `gui.triangles(a, b, c, color = 0xFFFFFF)`

9. `gui.rect(topleft, bottomright, radius = 1, color = 0xFFFFFF)`

* radius：画笔宽度

10. `gui.text(content, pos, font_size = 15, color = 0xFFFFFF)`

* pos：文本左上角

11. `ti.rgb_to_hex(rgb)`

```python
rgb = (0.4, 0.8, 1.0)
hex = ti.rgb_to_hex(rgb)  # 0x66ccff

rgb = np.array([[0.4, 0.8, 1.0], [0.0, 0.5, 1.0]])
hex = ti.rgb_to_hex(rgb)  # np.array([0x66ccff, 0x007fff])
```

## 事件处理

```python
# 事件类型
ti.GUI.RELEASE  # 键或鼠标释放
ti.GUI.PRESS    # 键或鼠标按下
ti.GUI.MOTION   # 鼠标移动或鼠标滚轮
```

```python
# 事件键
# for ti.GUI.PRESS and ti.GUI.RELEASE event:
ti.GUI.ESCAPE  # Esc
ti.GUI.SHIFT   # Shift
ti.GUI.LEFT    # Left Arrow
'a'            # 用小写字母表示字母表
'b'
...
ti.GUI.LMB     # 鼠标左键
ti.GUI.RMB     # 鼠标右键
# for ti.GUI.MOTION event:
ti.GUI.MOVE    # 鼠标移动
ti.GUI.WHEEL   # 鼠标滚轮滚动
```

```python
# 事件过滤器
# if ESC pressed or released:
gui.get_event(ti.GUI.ESCAPE)
# if any key is pressed:
gui.get_event(ti.GUI.PRESS)
# if ESC pressed or SPACE released:
gui.get_event((ti.GUI.PRESS, ti.GUI.ESCAPE), (ti.GUI.RELEASE, ti.GUI.SPACE))
```

1. `gui.running`

```python
while gui.running:
    if gui.get_event(ti.GUI.ESCAPE):
        gui.running = False

    render()
    gui.set_image(pixels)
    gui.show()
```

2. `gui.get_event(a, ...)`

从队列中取出事件，放在 gui.event 中。

```python
if gui.get_event():
    print('Got event, key =', gui.event.key)
```

3. `gui.get_events(a, ...)`

返回事件生成器，而不是将事件存储在 gui.event 中。

```python
for e in gui.get_events():
    if e.key == ti.GUI.ESCAPE:
        exit()
    elif e.key == ti.GUI.SPACE:
        do_something()
    elif e.key in ['a', ti.GUI.LEFT]:
        ...
```

4. `gui.is_pressed(key, ...)`

```python
while True:
    gui.get_event()  # must be called before is_pressed
    if gui.is_pressed('a', ti.GUI.LEFT):
        print('Go left!')
    elif gui.is_pressed('d', ti.GUI.RIGHT):
        print('Go right!')
```

5. `gui.get_cursor_pos()`

6. `gui.fps_limit`

返回最大 FPS，如果没有限制则返回 None，默认值为 60。

```python
gui.fps_limit = 24
```

## GUI 部件

1. `gui.slider(text, minimum, maximum, step=1)`

2. `gui.label(text)`

3. `gui.button(text, event_name=None)`

```python
radius = gui.slider('Radius', 1, 50)
while gui.running:
    print('The radius now is', radius.value)
    ...
    radius.value += 0.01
    ...
    gui.show()
```

## 图像 I/O

1. `ti.imwrite(img, filename)`

```python
import taichi as ti

ti.init()

shape = (512, 512)
type = ti.u8
pixels = ti.field(dtype=type, shape=shape)

@ti.kernel
def draw():
    for i, j in pixels:
        pixels[i, j] = ti.random() * 255    # integars between [0, 255] for ti.u8

draw()

ti.imwrite(pixels, f"export_u8.png")
```

```python
import taichi as ti

ti.init()

shape = (512, 512)
channels = 3
type = ti.f32
pixels = ti.Matrix.field(channels, dtype=type, shape=shape)

@ti.kernel
def draw():
    for i, j in pixels:
        for k in ti.static(range(channels)):
            pixels[i, j][k] = ti.random()   # floats between [0, 1] for ti.f32

draw()

ti.imwrite(pixels, f"export_f32.png")
```

2. `ti.imread(filename, channels=0)`

通道数默认为0，意味着对于图像文件是自适应的。返回值为 np.ndarray(dtype=np.uint8)。

3. `ti.imshow(img, windname)`

创建 ti.GUI 的实例以显示图像。

4. `ti.imresize(img, w, h=None)`

若 h 未被指定，默认等于 w。

## 零拷贝帧缓冲

`gui = ti.GUI(res, title, fast_gui=True)`

fast_gui 选项可以带来更好的性能，但仅能使用 gui.set_image 作为绘制函数。图像数据的格式必须为 ti.Vector(3)（rgb）或 ti.Vector(4)（rgba），通道的数据类型必须为 ti.f32、ti.f64 或 ti.u8。

