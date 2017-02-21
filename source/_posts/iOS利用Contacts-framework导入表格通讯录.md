---
title: iOS利用Contacts.framework导入表格通讯录
date: 2017-02-21 13:33:11
tags: ios,contacts.framework,excel,通讯录

---

#故事背景
公司有有个excel文件保存更新公司成员的通讯信息。每次修改都不知道改了哪里，而且手工一个一个往通讯录里面添加，两个就没有耐心了。因为有时候这些信息还真的会用到，为了用的时候不抓瞎，只能想办法导入到通讯录。
#过程
公司通讯录结构还是很复杂的，有姓名、电话、手机、邮箱、部门、职位。幸好这些在iphone系统通讯录里面都可以有。这个时候问题来了
excel文件ios不知道怎么处理？
我的处理办法是，首先把字段安排好。姓名、手机、邮箱、部门、职位。然后导出csv，这样就成了纯文本文件了。（我这里还做了一件事，打开csv。1.把没有用的空格去掉、2.把不和谐的换行和引号去掉了、3.重新保存成unicode编码的文件txl.csv）
然后其实就没有什么了，直接开始上代码

    func func3() -> () {
        let path = Bundle.main.path(forResource: "txl", ofType: "csv")
        do {
            let string = try String(contentsOfFile: path!)
            let lines = string.components(separatedBy: "\n")
            let request = CNSaveRequest()
            var lastDepartment = "";
            for line in lines {
                var labels = line.components(separatedBy: ",");
                let name = labels[0]
                let phone = labels[1]
                let email = labels[2]
                let department : String = labels[3]
                let title = labels[4]
                
                let lastName = name.substring(to: name.index(name.startIndex, offsetBy: 1))
                let givenName = name.substring(from: name.index(name.startIndex, offsetBy:1))
                
                let contact = CNMutableContact()
                contact.givenName = givenName
                contact.familyName = lastName
                
                let phonenumber = CNLabeledValue(label: CNLabelPhoneNumberMobile, value: CNPhoneNumber(stringValue: phone))
                contact.phoneNumbers = [phonenumber]
                
                let workEmail = CNLabeledValue(label: CNLabelWork, value: email as NSString)
                contact.emailAddresses = [workEmail]
                
                contact.organizationName = "公司名称"
                if department != "" {
                    contact.departmentName = department;
                    lastDepartment = department;
                }else{
                    contact.departmentName = lastDepartment;
                }
                
                contact.jobTitle = title;
                
                request.add(contact, toContainerWithIdentifier: nil)
            }
            
            let store = CNContactStore()
            
            do {
                try store.execute(request)
            } catch let e {
                print(e)
            }
            
        } catch let e {
            print(e)
        }
    }

导入到手机了。还是不行，因为我们是机械的新建联系人，致使现在通讯录里面的联系人出现了重复的现象。这个功能其实用代码规避一下也是可以的，但是我还是比较懒的。我下载了一个QQ同步助手，在这个app里面的通讯录管理里面有一个合并联系人。app可以自动合并一部分，自动合并不了的会提示手动合并。到此处才算是圆满结束。

#源码
[iOS利用Contacts.framework导入表格通讯录——Demo](https://github.com/caicai0/ios_demo/tree/master/contacts_demo "https://github.com/caicai0/ios_demo/tree/master/contacts_demo")

