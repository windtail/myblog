---
title: "体验 pyimgui"
date: 2024-02-19T22:09:53+08:00
tags:
  - python
  - imgui
draft: false
---

通过 `egui` 了解到了 [`imgui`](https://github.com/dearimgui) 这个库，这个库的想法是挺好的，很像我们在嵌入式系统中常用的界面显示方式，
即每隔一段时间，刷新下整个界面。没有状态同步，简单并且不容易出错。


`imgui` 这个库还有一个特点就是没有文档， [`pyimgui`](https://pyimgui.readthedocs.io/en/latest/) 略有一点文档

## 简单封装代码

```python
# coding: utf-8

from __future__ import annotations

import ctypes as ct
from pathlib import Path
from typing import Any, Callable, Optional, Set, Tuple, Union

import imgui  # type: ignore
import pyglet  # type: ignore
from attrs import define
from imgui.integrations.pyglet import create_renderer  # type: ignore
from PIL import Image
from pyglet import gl

__all__ = ["SimpleWin"]

_Render = Union[
    imgui.integrations.pyglet.PygletFixedPipelineRenderer,
    imgui.integrations.pyglet.PygletProgrammablePipelineRenderer,
]


@define
class SimpleWin:
    window: pyglet.window.Window
    impl: _Render

    @classmethod
    def create(
        cls,
        width: Optional[int] = None,
        height: Optional[int] = None,
        *,
        caption: Optional[str] = None,
        resizable: bool = True,
        fullscreen: bool = False,
        center: bool = True,
    ) -> SimpleWin:
        display = pyglet.canvas.Display()
        screen = display.get_default_screen()
        screen_width = screen.width
        screen_height = screen.height

        if width is None:
            win_width = screen_width
        else:
            win_width = min(width, screen_width)

        if height is None:
            win_height = screen_height
        else:
            win_height = min(height, screen_height)

        window = pyglet.window.Window(
            width=win_width,
            height=win_height,
            resizable=resizable,
            caption=caption,
            fullscreen=fullscreen,
        )

        if center and not fullscreen:
            window.set_location((screen_width - win_width) // 2, (screen_height - win_height) // 2)

        gl.glClearColor(1, 1, 1, 1)
        imgui.create_context()
        impl = create_renderer(window)

        return SimpleWin(window, impl)

    @staticmethod
    def default_screen_size() -> Tuple[int, int]:
        screen = pyglet.canvas.get_display().get_default_screen()
        return screen.width, screen.height

    @staticmethod
    def make_glyph_ranges(characters: Set[str]) -> Any:
        chars = sorted({ord(c) for c in characters})

        # 需要将连续的区域合成一个[begin,end]（注意：inclusive）的区间，最后一个结尾是0

        index_list = (
            [None] + [i for i in range(1, len(chars)) if chars[i] - chars[i - 1] > 1] + [None]
        )
        partitions = [chars[index_list[j - 1] : index_list[j]] for j in range(1, len(index_list))]

        def params():
            for p in partitions:
                yield p[0]
                yield p[-1]
            yield 0

        return imgui.GlyphRanges(list(params()))

    @staticmethod
    def make_glyph_ranges_from_file(path: str) -> Any:
        chars: Set[str] = set()

        src_path = Path(path)
        if src_path.is_dir():
            for f in src_path.rglob("*.py"):
                with open(f, "rt", encoding="utf-8") as fp:
                    chars = chars.union(fp.read())
        else:
            with open(src_path, "rt", encoding="utf-8") as fp:
                chars = chars.union(fp.read())

        return SimpleWin.make_glyph_ranges(chars)

    def load_font(self, font_path: Union[str, Path], font_size: int, glyph_ranges: Any) -> Any:
        io = imgui.get_io()

        path = font_path.as_posix() if isinstance(font_path, Path) else font_path

        font = io.fonts.add_font_from_file_ttf(path, font_size, glyph_ranges=glyph_ranges)
        self.impl.refresh_font_texture()
        return font

    def load_chs_font(
        self, font_path: Union[str, Path], font_size: int, full_chinese: bool = False
    ) -> Any:
        io = imgui.get_io()

        if full_chinese:
            glyph_ranges = io.fonts.get_glyph_ranges_chinese_full()
        else:
            glyph_ranges = io.fonts.get_glyph_ranges_chinese()

        return self.load_font(font_path, font_size, glyph_ranges)

    @staticmethod
    def load_image(
        image_path: Path, width: Optional[int] = None, height: Optional[int] = None
    ) -> object:
        image = Image.open(image_path)
        if width is not None and height is not None:
            image.resize((width, height))
        image_data = image.convert("RGBA").tobytes()
        w = image.width
        h = image.height
        image.close()

        texture = gl.GLuint()
        gl.glGenTextures(1, ct.pointer(texture))
        gl.glActiveTexture(gl.GL_TEXTURE0)
        gl.glBindTexture(gl.GL_TEXTURE_2D, texture)
        gl.glTexImage2D(
            gl.GL_TEXTURE_2D, 0, gl.GL_RGBA, w, h, 0, gl.GL_RGBA, gl.GL_UNSIGNED_BYTE, image_data
        )
        gl.glTexParameteri(gl.GL_TEXTURE_2D, gl.GL_TEXTURE_MAG_FILTER, gl.GL_LINEAR)
        gl.glTexParameteri(gl.GL_TEXTURE_2D, gl.GL_TEXTURE_MIN_FILTER, gl.GL_LINEAR)
        gl.glTexParameteri(gl.GL_TEXTURE_2D, gl.GL_TEXTURE_WRAP_S, gl.GL_CLAMP_TO_EDGE)
        gl.glTexParameteri(gl.GL_TEXTURE_2D, gl.GL_TEXTURE_WRAP_T, gl.GL_CLAMP_TO_EDGE)

        return texture

    @staticmethod
    def quit():
        pyglet.app.exit()

    def run(self, update: Callable[[], None]):
        def draw(_dt):
            self.impl.process_inputs()
            update()
            self.window.clear()
            imgui.render()
            self.impl.render(imgui.get_draw_data())

        pyglet.clock.schedule_interval(draw, 1 / 120.0)
        pyglet.app.run()
        self.impl.shutdown()
```

代码解释如下：

1. backend: imgui支持很多backend，不知道选哪一个好，我这里选了pyglet，这个看起来更好跨平台，并且还挺简单的
2. 字体: imgui的字体需要生成texture放到内存里才能使用，英文字母很少，生成字体很快。但是中文字特别多，如果字号较多，并且需要支持汉字的话，
   就比较多了，初始化时会有较明显的卡顿，解决方案有几种：
   - 只支持常用汉字，也就是 `io.fonts.get_glyph_ranges_chinese()`
   - 只支持程序中的汉字，可以扫描源代码中的汉字，然后生成自定义的glyph_ranges，参见代码中的 `make_glyph_ranges`，这个功能 `pyimgui` 文档里没有明确地写出来
   - cache生成的texture到文件中，下次程序运行时直接加载，这个可能需要仔细了读下源代码了，暂时没有了解
3. 图片: 图片也要先生成texture才能被使用，并且 `imgui` 没有提供函数，需要参考backend来实现，参见代码中的 `load_image`
4. 按键处理: 按键处理不是很完善，当前focus显示不同的样式，也不太好处理。使用C++可以用一些内部函数来实现，但是 `pyimgui` 并没有把这些内部的api开出来。   
   控件要先render，之后才能判断item是有focus，这就是一个鸡生蛋，蛋生鸡的问题。
5. 自定义样式: 可以使用预设置的主题，例如 `imgui.style_colors_dark()`，之后还可以对样式进行微调，例如：
   ```python
   style = imgui.get_style()
   style.frame_padding = (4.0, 3.0)
   style.frame_rounding = 0.0   
   style.colors[imgui.COLOR_NAV_HIGHLIGHT] = imgui.Vec4(1, 1, 1, 1)
   ```   
6. 自定义控件: 我们可以使用一些 `draw_list` 的api来完成自定义控件，但复杂控件的自定义有困难，比如我们想修改一下 `TextEdit`，
   这几乎是不可能的。但简单的控件还是可以绘制的，例如：    
   ```python
    def toggle_button(
        str_id: str,
        state: bool,
        rect: Rect,
        ro: bool = False,
        on_color: Optional[Color] = None,
        off_color: Optional[Color] = None,
    ) -> Tuple[bool, bool]:
        on_color = on_color or Color.green
        off_color = off_color or Color.tint_gray
    
        p = imgui.get_cursor_screen_pos()
        draw_list = imgui.get_window_draw_list()
        size = rect.size
        diameter = max(4, min((size.width - 4) // 2, imgui.get_frame_height(), size.height))
        width = min(size.width, 2 * diameter + 5)
        height = diameter
        rounding = 0.4
    
        clicked = False
        new_state = state
    
        # 缩小实际控件的区域，并映射到窗口坐标系
        center_x = p.x + size.width / 2
        center_y = p.y + size.height / 2
    
        left = center_x - width / 2
        right = left + width - 1
        top = center_y - height / 2
        bottom = top + height - 1
    
        x, y = imgui.get_cursor_pos()
        x += left - p.x
        y += top - p.y
        imgui.set_cursor_pos((x, y))
    
        imgui.invisible_button(str_id, width, height)
        if not ro and imgui.is_item_clicked():
            clicked = True
            new_state = not new_state
    
        clr = on_color if new_state else off_color
        draw_list.add_rect_filled(
            left, top, right, bottom, imgui.get_color_u32_rgba(*clr.rgbaf()), height * rounding
        )
    
        clr = Color.white
        if new_state:
            draw_list.add_rect_filled(
                left,
                top,
                left + diameter - 1,
                top + diameter - 1,
                imgui.get_color_u32_rgba(*clr.rgbaf()),
                height * rounding,
            )
        else:
            draw_list.add_rect_filled(
                right + 1 - diameter,
                bottom + 1 - diameter,
                right,
                bottom,
                imgui.get_color_u32_rgba(*clr.rgbaf()),
                height * rounding,
            )
    
        return clicked, new_state
   ```
   其中 `Rect` 和 `Color` 是两个简单的类，可以使用 `Tuple` 来代替。如果只想在现在控件上加点东西，
   也可以直接在控件位置上使用 `front draw list` 来绘制一些东西，但是效果很一般。
   
## 总结

对于不了解 PyQt 的同志，建议先学 PySide2 再来写界面，对于已经了解 PyQt 的同志，直接用 PyQt 就可以了。
我感觉 `pyimgui` 的好处就是打包后，体积比较小，安装更方便，其他也没啥好处了。

顺带提一下，B站上有一个哥们讲了一点 imgui ，对我有一些帮助，大家可以去搜下看见，就是那个说话很狂的哥们 ^_^
