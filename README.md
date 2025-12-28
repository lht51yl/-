# -
学术写作、规范与伦理
[sagnac.py](https://github.com/user-attachments/files/24359575/sagnac.py)
from splayout import *
import numpy as np
import os
import sys
import math

# 创建单元
cell = Cell("STRAIGHT_DIRECTIONAL_COUPLER_WITH_SBEND_AND_HALFCIRCLE")
wg_layer = Layer(1, 0)


def draw_straight_directional_coupler(cell, layer,
                                      center=(0, 0),
                                      wg_width=1.3,
                                      gap=0.3,
                                      coupling_length=10,
                                      input_output_length=50):
    """
    绘制直波导型定向耦合器

    参数:
    - center: 耦合器中心坐标
    - wg_width: 波导宽度 (um)
    - gap: 耦合间隙 (um)
    - coupling_length: 耦合长度 (um)
    - input_output_length: 输入输出波导长度 (um)
    """
    cx, cy = center

    # 计算波导位置
    upper_wg_y = cy + gap / 2 + wg_width / 2
    lower_wg_y = cy - gap / 2 - wg_width / 2

    # 1. 绘制上波导的输入输出
    # 上波导输入
    upper_input_start = Point(cx - coupling_length / 2 - input_output_length, upper_wg_y)
    upper_input_end = Point(cx - coupling_length / 2, upper_wg_y)
    upper_input_wg = Waveguide(upper_input_start, upper_input_end, width=wg_width)
    upper_input_wg.draw(cell, layer)

    # 上波导输出
    upper_output_start = Point(cx + coupling_length / 2, upper_wg_y)
    upper_output_end = Point(cx + coupling_length / 2 + input_output_length, upper_wg_y)
    upper_output_wg = Waveguide(upper_output_start, upper_output_end, width=wg_width)
    upper_output_wg.draw(cell, layer)

    # 2. 绘制下波导的输入输出
    # 下波导输入
    lower_input_start = Point(cx - coupling_length / 2 - input_output_length, lower_wg_y)
    lower_input_end = Point(cx - coupling_length / 2, lower_wg_y)
    lower_input_wg = Waveguide(lower_input_start, lower_input_end, width=wg_width)
    lower_input_wg.draw(cell, layer)

    # 下波导输出
    lower_output_start = Point(cx + coupling_length / 2, lower_wg_y)
    lower_output_end = Point(cx + coupling_length / 2 + input_output_length, lower_wg_y)
    lower_output_wg = Waveguide(lower_output_start, lower_output_end, width=wg_width)
    lower_output_wg.draw(cell, layer)

    # 3. 绘制耦合区域
    # 上耦合波导
    upper_coupler_start = Point(cx - coupling_length / 2, upper_wg_y)
    upper_coupler_end = Point(cx + coupling_length / 2, upper_wg_y)
    upper_coupler_wg = Waveguide(upper_coupler_start, upper_coupler_end, width=wg_width)
    upper_coupler_wg.draw(cell, layer)

    # 下耦合波导
    lower_coupler_start = Point(cx - coupling_length / 2, lower_wg_y)
    lower_coupler_end = Point(cx + coupling_length / 2, lower_wg_y)
    lower_coupler_wg = Waveguide(lower_coupler_start, lower_coupler_end, width=wg_width)
    lower_coupler_wg.draw(cell, layer)

    # 4. 添加标注文本（可选）
    print(f"直波导定向耦合器绘制完成")
    print(f"参数: 波导宽度={wg_width}um, 耦合间隙={gap}um, 耦合长度={coupling_length}um")

    # 返回关键点坐标，便于后续连接
    return {
        'upper_input': upper_input_start,
        'upper_output': upper_output_end,
        'lower_input': lower_input_start,
        'lower_output': lower_output_end,
        'upper_output_start': upper_output_start,  # 添加输出起始点
        'lower_output_start': lower_output_start  # 添加输出起始点
    }


def add_sbends_to_outputs(cell, layer, coupler_points, wg_width=1.3):
    """
    在定向耦合器的输出波导上添加S弯曲
    使用简单的起点和终点定义方式

    参数:
    - coupler_points: 定向耦合器的关键点
    - wg_width: 波导宽度
    """

    # 上输出波导的S弯曲
    upper_start_point = coupler_points['upper_output']
    upper_end_point = Point(upper_start_point.x + 200, upper_start_point.y + 150)
    upper_sbend = ASBend(upper_start_point, upper_end_point, width=wg_width)
    upper_sbend.draw(cell, layer)

    # 下输出波导的S弯曲
    lower_start_point = coupler_points['lower_output']
    lower_end_point = Point(lower_start_point.x + 200, lower_start_point.y - 150)
    lower_sbend = SBend(lower_start_point, lower_end_point, width=wg_width)
    lower_sbend.draw(cell, layer)

    print("S弯曲添加完成")

    return {
        'upper_sbend_end': upper_end_point,
        'lower_sbend_end': lower_end_point
    }


def add_half_circle_connection(cell, layer, sbend_points, wg_width=1.3, radius=150):
    """
    在两个S弯曲末端之间添加半圆连接

    参数:
    - sbend_points: S弯曲的端点
    - wg_width: 波导宽度
    - radius: 半圆半径
    """

    # 获取S弯曲的末端点
    upper_end = sbend_points['upper_sbend_end']
    lower_end = sbend_points['lower_sbend_end']

    # 计算半圆中心点
    center_x = upper_end.x
    center_y = (upper_end.y + lower_end.y) / 2

    # 绘制上半圆（从上方S弯曲末端开始）
    upper_half_circle_center = Point(center_x, center_y)
    upper_half_circle_start_angle =1.5* math.pi  # 180度
    upper_half_circle_end_angle = 2.5*math.pi  # 0度
    upper_half_circle = Bend(upper_half_circle_center, upper_half_circle_start_angle,
                             upper_half_circle_end_angle, wg_width, radius)
    upper_half_circle.draw(cell, layer)




    print(f"半圆连接添加完成，半径={radius}um")


# 绘制直波导定向耦合器
coupler_points = draw_straight_directional_coupler(
    cell, wg_layer,
    center=(100, 0),
    wg_width=1.3,
    gap=0.3,
    coupling_length=10,
    input_output_length=50
)

# 在输出波导上添加S弯曲
sbend_points = add_sbends_to_outputs(
    cell, wg_layer,
    coupler_points,
    wg_width=1.3
)

# 在两个S弯曲末端之间添加半圆连接
add_half_circle_connection(
    cell, wg_layer,
    sbend_points,
    wg_width=1.3,
    radius=150.8
)

print("带有S弯曲输出和半圆连接的直波导定向耦合器绘制完成")

# 生成GDS文件
filename = "sganac.gds"
current_dir = os.path.dirname(os.path.abspath(__file__)) if '__file__' in locals() else os.getcwd()
output_path = os.path.join(current_dir, filename)
make_gdsii_file(output_path)

print(f"GDS文件已保存至：{output_path}")
