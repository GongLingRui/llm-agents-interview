# 查看视频音频的信息

ffmpeg -i video.mp4

-hide_banner没有用的信息给隐藏掉的

mp3的查看方式：

# 格式转换

mp4/avi/mov/mkv/flv/m4v

方式1:重新编码转换

ffmpeg -i 要转换1的视频地址 转换后的视频存放地址

源文件地址 :/Users/Alpha/Desktop/new.mp4

转换为flv

转换后的地址：

/Users/Alpha/Destop/new.flv

方式2:不编码直接转换（速度相对快一些）


ffmpeg -i 要转换的视频地址 -c copy 转换后的视频地址

转换后的地址:/Users/Alpha/Desktop/new_nocodec.flv

ffmpeg -i 要转换的文件 转换后的文件的地址（文件后缀非常重要的）

ffmpeg - i /Users/Alpha/Destop /new.mp4 /Users/Alpha/Destop/new.flv

# 从视频当中提取音频

方式1:自动编码提取：

ffmepg -i 视频文件地址 要输出的音频文件地址

视频文件地址 ：/Users/Alpha/Desktop/new.mp4

要输出的音频文件路线：/Users/Alpha/Destop/test.mp3

ffmpeg -i /Users/Alpha/Destop/new.mp4 /Users/Alpha/Destop/test.mp3

# 快速截断视频片段

ffmpeg -ss 00:00:05 -t 5 -i 源文件地址  截取后的要输出的路径的地址

ffmpeg -ss 00:00:10 -t 5 -i /Users/Alpha/Destop/new.mp4 /Users/Alpha/Destop/new_cut.mp4

方式2:

ffmpeg -ss 00:00:20 -to 00:00:20 -i /Users/Alpha/Destop/new.mp4 /Users/Alpha/Destop/new_cut.mp4


# 三种方式更改视频分辨率



### 缩放视频的3种需求
缩放视频一般有3种需求：
1. 自定义分辨率
2. 等比例更改分辨率
3. 按倍数更改分辨率




### 对应命令
1. 自定义更改分辨率 命令
```
ffmpeg -i 要缩放的视频 -vf scale=720:720 缩放后的视频存储路径
```
（宽：高）


2. 等比例更改分辨率 命令
```
fmeg -i 要缩放的视频 -vf scale=960:-1 缩放后的视频存储路径
```
（负1代表等比例）
注：此处“fmeg”应为“ffmpeg”（命令工具正确名称）


3. 按倍数更改分辨率 命令
```
fmeg -i 要缩放的视频 -vf scale=iw/2:ih/2 缩放后的视频存储路径
```
（iw原视频的宽, ih原视频的高）
注：此处“fmeg”应为“ffmpeg”（命令工具正确名称）


### 原始视频路径
/Users/Alpha/Desktop/animal.mp4

ffmpeg -i /Users/Alpha/Desktop/animal.mp4 -vf scale=960:-1 /Users/Alpha/Desktop/animal_scale.mp4

ffmpeg -i /users/alpha/desktop/animal.mp4 -vf scale=720:720 /users/alpha/desktop/animal_scale.mp4

# 按照倍数更改分辨率

ffmpeg -i /Users/Alpha/Desktop/animal.mp4 -vf scale = iw/2:ih /2 /Users/Alpha/Desktop/animal_scale.mp4

# 删除视频当中的音频

ffmpeg -i 原始视频的路径 -c:v copy -an 输出视频路径

ffmpeg -i /users/alpha/desktop/test.mp4 
-c:v copy -an /users/alpha/desktop/test_no_audio.mp4

-c:v copy不要直接编码视频直接复制就可以了

# 在python当中使用ffmpeg

```
import subprocess

# 在python当中使用ffmpeg
def start08():
    # ffmpeg 原始命令
    command ='ffmpeg -i /Users/Alpha/Desktop/test.mp4 -c:v copy -an /Users/Alpha/Desktop/test_no_audio.mp4'
    print(f'启用ffmpeg命令：{command}')
    # 通过subprocess调用系统命令提示行
    subprocess.run(command,shell=True,stderr=subprocess.PIPE,stdout =subprocess.DEVNULL)
    print('命令执行完毕')
if __name__ == '__main__':

```
# 显示任务进度条

```
from better_ffmpeg_progress import FfmpegProgress

pip3 intsall better_ffmpeg_progress

ffmpeg原始命令，每10真截取一张图片

command = f'ffmpeg -i {input_video_path} -r 10 -f image2 {out_image_path}'

command_list =command.split(' ')

# 执行ffmpeg命令，输出执行进度

progress = FfmpegProgress(command_list)

progress.run_command()
```

# ffmpeg 去除视频水印

ffmpeg -i 原始视频 -filter_complex delogo=x=732:y=21:w=211:h=65:show=0 输出无水印视频
# ffmpeg 去水印命令

ffmpeg -i y原始视频 -filter_complex delogo = x=732:y=211:h=65:show=0 输出无水印的视频

难点：水印坐标（x,y),水印的长宽(w,h)无法准确测量

需要新安装一个python的包

pip3 install opencv-python

delogo_x='732'

delogo_y='21'

w='211'


# 获取水印的位置，返回水印的x,y,w,h

def getroi(name):

    print('获取水印的位置)
    video = cv2.VideoCapture(name)
    success, frame = video.read()
    roi = select_roi(frame, '请用鼠标狂炫水印位置，按enter健退出')
    video.release()
    cv2.destroyAllWindows()
    print(roi)
    return roi


# 全自动去水印（终极版本）

ffmpeg 去水印命令

原始视频 -filter_complex delogo = x=732:y=21:w=211:h=65:show=0 输出无水印

水印x周，水印y周 水印宽 水印高

1.自动下载视频

2准备原始的水印图片

3.自动定位水印的位置

```
def getwaterLoc(video,img):
# 定位水印位置分为两个步骤：
# 计算原始水印和视频分辨率的对应关系，不同的视频分辨率需要对水印图片进行换算处理
# 横屏和竖屏的视频，也需要进行不同的计算处理
 if video.width <video_height:
    # 横屏和竖屏水印缩放不一样的
    scale = video_size / 1920
else:
    scale = video_size / 1000

adjust_w = 0
adjust_h = int(40*scale)

# 第二部：使用处理后的图片定位水印在视频当中的位置

res = cv2.matchTemplate(img, template, method)

min_val,max_val,min_loc,max_loc = cv2.minMaxLoc(res)

# 旋转视频（横屏转竖屏）

ffmpeg -i input.mp4 -vf transpose =1 output.mp4

0 逆时针旋转90度的并且垂直旋转

1:顺势站旋转90度的

2逆时针旋转90度的

3顺时针旋转90度以后并且垂直翻转

ffmpeg -i {input_video} -vf transpose ={angle} {output_video}`

# 视频转图片｜图片转视频

ffmpeg -i input_video -t 22 -r 10 3%.jpg

图片转视频的命令

ffmpeg -r 15 -i %3d.jpg output_video

ffmpeg -i {input_video} -r 10 {os.path.join(folder,"3d.jpg")}"

# 给视频添加字幕

ass字幕文件

ffmpeg -i input_video -filter_complex ass=subtitle.ass output_video

ffmpeg -i input_video -filkter_complex ass=subtitle.ass output_video


# ffmpeg 给视频添加水印

ffmpeg - i input.mp4(原始视频) -i watermakr.png(水印图片) -filter_complex overlay = x=0:y=0(水印在视频当中的位置) output.mp4(输出视频)


