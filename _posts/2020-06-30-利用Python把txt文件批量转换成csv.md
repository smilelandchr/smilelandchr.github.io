---
layout:     post
title:         2020-06-30-利用Python把txt文件批量转换成csv
date:        2020-06-30
author:     SMILELAND
categories: 做文章
catalog: true
tags:
    - python
    - txt
    - csv
---

经常会对pscad仿真的数据利用matlab进行二次分析，而pscad的数据导出为.txt文件，而matlab则需要csv文件进行导入操作，故会有大量的将txt转换为csv的需求。

对于简单的数据处理，直接利用excel自带的从txt导入数据就可以，但是若需要大量进行此类操作就比较麻烦。在网上找了一段代码，原有代码中的数据分隔用的是";"，而pscad的导出文件用的是","，简单修改一下，亲测好用。

引用自：
https://zhuanlan.zhihu.com/p/86456721

``` python
import csv
import os
import shutil
from chardet.universaldetector import UniversalDetector


def get_encode_info(file):
    with open(file, 'rb') as f:
        detector = UniversalDetector()
        for line in f.readlines():
            detector.feed(line)
            if detector.done:
                break
        detector.close()
        return detector.result['encoding']


def read_file(file):
    with open(file, 'rb') as f:
        return f.read()


def write_file(content, file):
    with open(file, 'wb') as f:
        f.write(content)


def convert_encode2utf8(file, original_encode, des_encode):
    file_content = read_file(file)
    file_decode = file_content.decode(original_encode, 'ignore')
    file_encode = file_decode.encode(des_encode)
    write_file(file_encode, file)


## Move *.txt to a folder
def move2txtfolder(path, txt_file_list):
    txt_folder_path = path + '\\txt'
    if not os.path.exists(txt_folder_path):
        os.makedirs(txt_folder_path)

    for file in txt_file_list:
        des_path = os.path.join(txt_folder_path, os.path.basename(file))
        shutil.move(file, des_path)


##在路径中找出所有的*.txt文件
def findtxt(path, txt_file_list):
    file_name_list = os.listdir(path)
    for filename in file_name_list:
        de_path = os.path.join(path, filename)
        if os.path.isfile(de_path):
            if de_path.endswith(".txt"):  # Specify to find the txt file.
                txt_file_list.append(de_path)
        else:
            findtxt(de_path, txt_file_list)


def txt2csv(txt_file):
    ##先把所有文件的encoding都转换成utf-8
    encode_info = get_encode_info(txt_file)
    if encode_info != 'utf-8':
        convert_encode2utf8(txt_file, encode_info, 'utf-8')

    csv_file = os.path.splitext(txt_file)[0] + '.csv'
    with open(csv_file, 'w+', newline='', encoding='utf-8') as csvfile:
        writer = csv.writer(csvfile, dialect='excel')

        with open(txt_file, 'r', encoding='utf-8') as txtfile:
            for line in txtfile.readlines():
                line_list = line.strip('\n').split(',')
                writer.writerow(line_list)


if __name__ == '__main__':
    folder_path = r'E:\OneDrive - zju.edu.cn\Simulation Models\2. PSCAD46\5. 为Tensorflow出仿真数据\仿真数据'
    # ##如果文件夹中还有子文件夹，请用findtxt函数
    # txt_file_list = []
    # findtxt(folder_path, txt_file_list)

    ##如果文件夹中没有子文件夹的时候直接使用推导式来生产txt文件的list
    txt_file_list = [os.path.join(folder_path, file) for file in os.listdir(folder_path) if
                     os.path.join(folder_path, file).endswith('.txt')]

    for txt_file in txt_file_list:
        txt2csv(txt_file)

    move2txtfolder(folder_path, txt_file_list)
```

主要实现功能如下：

1. 读取文件夹中所有txt文件，保存到list中
2. 针对每个txt文件，自动生产同文件名的csv文件
3. 对每个txt文件，根据分隔符来保存为csv文件，分隔符为分号“,”，在转换之前先把文件编码统一成'utf-8'，因为在实现过程中，发现总会有编码报错问题出现
4. 新建txt文件夹来存放所有txt文件

![转换后的文件](https://i.loli.net/2020/06/30/iw6UB4WhmzGE1O5.png)