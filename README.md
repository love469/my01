# my01
Image-to-text
这段代码的作用是从一张或者多张图片中提取文字，然后将需要的文字复制到特定网页的代码。
用于需要重复输入的相同格式但内容不同的图片里面的文字进行提取并复制黏贴到制定的网页和位置。
可以用扫描仪（相机）等工具对证件照进行处理，然后进入本代码进行中文识别，并把相应的文字复制黏贴到指定网站。
以下是代码：
注意：这段代码需要一些必要的依赖
tkinter
paddleOCR
 运行环境：python

——————————————————————————————————————————————————————————————————————————

import tkinter as tk
from tkinter import messagebox
from tkinterdnd2 import DND_FILES, TkinterDnD  # 引入拖拽库
from paddleocr import PaddleOCR
import os
import re

# 初始化 OCR
ocr = PaddleOCR()

# 默认姓名、地址和身份证号码变量
name = ''
address = ''
id_number = ''
image_file = ''  # 默认图片文件路径为空

# 进行OCR识别的方法
def run_ocr():
    global image_file, name, address, id_number
    # 每次识别前重置结果
    name = ''
    address = ''
    id_number = ''
    
    # 检查图片路径是否存在
    if os.path.exists(image_file):
        results = ocr.ocr(image_file, cls=True)
        print('识别结果:', results)  # 输出整个结果进行调试

        # 检查结果是否为 None
        if results is not None:
            # 解析OCR结果
            for line in results:
                for word in line:
                    text_line = word[-1]
                    text = text_line[0]
                    print('识别文本:', text)  # 输出文本调试

                    # 提取身份证号码逻辑
                    if "公民身份号码" in text:
                        continue
                    else:
                        id_match = re.search(r'\d{17}[\dXx]', text)  # 寻找身份证号码
                        if id_match:
                            id_number = id_match.group()  # 只保留数字部分

                    # 将识别文本中的“姓名”去掉
                    if "姓名" in text:
                        name = text.replace("姓名", "").strip()  # 去掉“姓名”
                    # 将识别文本中的“住址”去掉，并合并地址信息
                    elif "住址" in text:
                        address += text.replace("住址", "").strip() + " "  # 去掉“住址”并合并
                    elif "省" in text or "市" in text or "区" in text:  # 处理地址信息
                        address += text.strip() + " "  # 添加其他地址信息

            # 去掉地址末尾的空格
            address = address.strip()
            # 更新 GUI 中的变量值
            name_var.set(name)
            address_var.set(address)
            id_var.set(id_number)
        else:
            messagebox.showerror("错误", "OCR 识别失败，结果为空！")
    else:
        messagebox.showerror("错误", f"文件 {image_file} 不存在！")

# 创建GUI界面
def copy_to_clipboard(text):
    root.clipboard_clear()
    root.clipboard_append(text)
    messagebox.showinfo("复制成功", "内容已复制到剪贴板")

def clear_fields():
    name_var.set('')
    address_var.set('')
    id_var.set('')

def minimize_window():
    root.iconify()

def close_window():
    root.destroy()

# 更新图片文件路径
def update_image_file(path):
    global image_file
    image_file = path  # 设置路径
    path_entry.delete(0, tk.END)  # 清空输入框
    path_entry.insert(0, path)  # 插入新路径

# 拖放文件的处理
def drop(event):
    filepath = event.data  # 获取拖放的文件路径
    update_image_file(filepath)

# 创建带拖放功能的应用程序主窗口
root = TkinterDnD.Tk()
root.title("OCR 识别结果")
root.geometry("400x400")  # 增加高度以容纳新控件
root.configure(bg="#C81D25")  # 设置窗口背景为中国红

# 定义 StringVars
name_var = tk.StringVar(value=name)
address_var = tk.StringVar(value=address)
id_var = tk.StringVar(value=id_number)

# 姓名行
tk.Label(root, text="姓名:", bg="#C81D25", fg="white", font=("Arial", 12)).grid(row=0, column=0, padx=10, pady=5)
tk.Entry(root, textvariable=name_var, state='readonly').grid(row=0, column=1, padx=10, pady=5)
tk.Button(root, text="复制", command=lambda: copy_to_clipboard(name_var.get())).grid(row=0, column=2, padx=10, pady=5)

# 地址行
tk.Label(root, text="地址:", bg="#C81D25", fg="white", font=("Arial", 12)).grid(row=1, column=0, padx=10, pady=5)
tk.Entry(root, textvariable=address_var, state='readonly').grid(row=1, column=1, padx=10, pady=5)
tk.Button(root, text="复制", command=lambda: copy_to_clipboard(address_var.get())).grid(row=1, column=2, padx=10, pady=5)

# 身份证号码行
tk.Label(root, text="身份证号码:", bg="#C81D25", fg="white", font=("Arial", 12)).grid(row=2, column=0, padx=10, pady=5)
tk.Entry(root, textvariable=id_var, state='readonly').grid(row=2, column=1, padx=10, pady=5)
tk.Button(root, text="复制", command=lambda: copy_to_clipboard(id_var.get())).grid(row=2, column=2, padx=10, pady=5)

# 图片路径输入行
tk.Label(root, text="请输入图片路径:", bg="#C81D25", fg="white", font=("Arial", 12)).grid(row=3, column=0, padx=10, pady=5)
path_entry = tk.Entry(root, width=40)
path_entry.grid(row=3, column=1, padx=10, pady=5)

# 将输入框设置为可以接受拖放文件
path_entry.drop_target_register(DND_FILES)
path_entry.dnd_bind('<<Drop>>', drop)

# 启动识别按钮
tk.Button(root, text="启动识别", command=lambda: [update_image_file(path_entry.get()), run_ocr()]).grid(row=3, column=2, padx=10, pady=5)

# 按钮行
tk.Button(root, text="清空", command=clear_fields).grid(row=4, column=0, padx=10, pady=5)
tk.Button(root, text="最小化", command=minimize_window).grid(row=4, column=1, padx=10, pady=5)
tk.Button(root, text="关闭", command=close_window).grid(row=4, column=2, padx=10, pady=5)

# 添加底部说明文字（使用grid）
footer = tk.Label(root, text="啄木鸟客栈十八子", bg="#C81D25", fg="white", font=("Arial", 12))
footer.grid(row=5, column=0, columnspan=3, pady=10)  # 设置在最后一行，跨三列

root.mainloop()  # 运行GUI
