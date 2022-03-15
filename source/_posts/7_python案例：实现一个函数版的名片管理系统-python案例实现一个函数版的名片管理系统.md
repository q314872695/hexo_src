---
title: python案例：实现一个函数版的名片管理系统
tags: python
abbrlink: affd80a3
date: 2019-08-17 15:24:00
---

本案例使用了自定义函数以及对字符串的常见操作、判断语句和循环语句等知识。

# 要求
 1. 必须使用自定义函数，完成对程序的模块化。
 2. 名片信息至少包括：姓名、电话、住址。
 3. 必须完成的功能：增、删、改、查、退出。
# 分析
 1. 首先呢，我们应该先定义一个全局变量，用于存储所有的名片信息。
```python
cards = [{
    "name": "张三",
    "phone": "10086",
    "address": "山西省",
}, {
    "name": "李四",
    "phone": "10010",
    "address": "北京市",
}]  # 定义一个的列表用于存放名片信息，默认里面有张三和李四的信息，方便以后调试用。
```
2. 完成增加，删除，修改，查找等相关操作的函数（cards是可变类型的全局变量，故在函数中不用加global也可调用），注意：如定义变量 a: int = 0, int只是单纯的提示开发人员它的类型是int,方便调试。
```python
def print_menu():
    """"完成打印功能菜单"""
    print("=" * 20)
    print("    名片管理系统")
    print(" 1:添加一个名片")
    print(" 2:删除一个名片")
    print(" 3:修改一个名片")
    print(" 4:查询一个名片")
    print(" 5:显示所有的名片")
    print(" 6:退出")
    print("=" * 20)


def add_card():
    """完成添加一个名片的功能"""
    new_infor: dict = {
    	"name": input("请输入一个名字："), 
    	"phone": input("请输入一个电话："), 
    	"address": input("请输入一个地址：")
    }
    cards.append(new_infor)
    print("添加成功！")


def delete_card():
    del_name = input("请输入要删除的名字：")
    for person in cards:
        if del_name == person["name"]:
            cards.remove(person)
            print("删除成功！")
            break
    else:
        print("找不到要删除的人！")


def update_card():
    name: str = input("请输入要修改的名字（只能通过名字来修改电话和住址）：")
    for person in cards:
        if name == person["name"]:
            phone = input("请输入新的的电话（直接回车则不修改）：")
            address = input("请输入新的的地址（直接回车则不修改）：")
            if phone:
                person["phone"] = phone
            if address:
                person["address"] = address
            print("修改成功")
            break
    else:
        print("找不到要修改的人！")


def find_card():
    """用来查询一个名片"""

    find_name: str = input("请输入要查询的名字（支持模糊查询）：")
    flag: int = 1
    for temp in cards:
        # 遍历名片中的所有名字，判断要查找的名字是否存在，不存在则打印查无此人
        if temp["name"].find(find_name) != -1:
            print("%s\t%s\t%s" % (temp["name"], temp["phone"], temp["address"]))
            flag = 0
    if flag:
        print("查无此人")


def show_all():
    print("姓名\t电话\t住址")
    for temp in cards:
        print("%s\t%s\t%s" % (temp["name"], temp["phone"], temp["address"]))
```
3. 最后完成主函数的功能，并调用主函数
```python
def main():
    """"完成对整个程序的控制"""
    # 打印功能提示
    print_menu()
    while True:
        # 获取用户的选择
        num: str = input("请输入功能序号：")
        # 判断输入的是否为数字
        if not num.isdigit():
            print("请输入数字！")
            continue
        # 转换成数字类型
        num: int = int(num)
        # 增
        if num == 1:
            add_card()
        # 删
        elif num == 2:
            delete_card()
        # 改
        elif num == 3:
            update_card()
        # 查
        elif num == 4:
            find_card()
        elif num == 5:
            show_all()
        elif num == 6:
            break
        else:
            print("请按号输入！")
        print()

# 调用主函数
if __name__ == '__main__':
    main()
```
