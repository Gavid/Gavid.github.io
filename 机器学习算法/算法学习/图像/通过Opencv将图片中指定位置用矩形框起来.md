**通过Opencv将图片中指定位置用矩形框起来**



* 通过对数据集的分析，初步实现了通过图片中文字的位置信息，将文字的位置用矩形进行框起来，便于后期进行文本的识别
* 代码：
```python
import os
import cv2

def reading_file_dirs():
    # 文件路径
    file_dirs = "../databases/train/"
    image_dirs = []
    text_dirs = []
    for file_dir in os.listdir(file_dirs):
        if file_dir[-3:] == 'jpg':
            image_dirs.append(file_dir)
        else:
            text_dirs.append(file_dir)
    # print(len(image_dirs))
    # print(len(text_dirs))
    # print(len(os.listdir(file_dirs)))
    # print(image_dirs[1][0:-4])

    image = cv2.imread(file_dirs+image_dirs[0])
    text_dir = file_dirs+image_dirs[0][0:-4]+".txt"
    data = []
    for line in open(text_dir, "r"):  # 设置文件对象并读取每一行文件
        data.append(line.split(',')[0:4])  # 将每一行文件加入到list中
    first_point = (int(data[0][0]),int(data[0][1]))
    last_point = (int(data[0][2]),int(data[0][3]))
    # 给image 中指定位置进行矩形的标注
    cv2.rectangle(image,first_point,last_point,(0,255,0),20)
    # 由于远程运行程序，尚未获取"X server"的权限， 所以图片的处理结果只能通过生成图片文件进行查看
    # cv2.imshow('demo',image)
    cv2.imwrite('../result_demo/demo.png',image)
```