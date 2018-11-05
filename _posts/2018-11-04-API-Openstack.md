---
layout: post
title: "Làm Việc Với API Openstack"
tags: Openstack
description: Làm Việc Với API Openstack
comments: true
---

# Giới thiệu 

Khái niệm: Một giao diện lập trình ứng dụng (tiếng anh Application Programming Interface hay API) là một giao diện mà một hệ thống máy tính hay ứng dụng cung cấp để cho phép các yêu cầu dịch vụ có thể được tạo ra từ các chương trình máy tính khác, và hoặc cho phép dữ liệu có thể được trao đổi qua lại giữa chúng. Cũng giống như bàn phím là một thiết bị giao tiếp giữa người dùng và máy tính.

# API trong Openstack 

Sử dụng API của Openstack có thể tạo máy ảo, tạo image và các hoạt động khác trong Openstack.

Để gửi các yêu cầu đến API, ta có thể sử dụng các cách thức sau: 

- cURL: là một công cụ dòng lệnh mà cho phép bạn gửi các yêu cầu  và nhận phản hồi theo http.

- Openstack command-line clients: Mỗi một project của Openstack cung cấp một command-line client mà cho phép bạn truy xuất vào API một cách dễ dàng. 

- REST clients: Cả Mozilla và Google đều cung cấp giao diện đồ họa dựa trên trình duyệt cho REST. 

- Openstack Python Software Development Kit (SDK): Sử dụng SDK để viết một script tự động Python mà tạo và quản lý tài nguyên trong cloud Openstack. SDK triển khai Python liên kết với Openstack API, nó cho phép bạn thực hiện các nhiệm vụ tự động trong python bởi gọi đến các objects python hơn là gọi đến REST trực tiếp. Tất cả các tools được triển khai bởi sử dung Python SDK. 

# Sử dụng Advanced RESTClient Chrome tác động đến API. 

### Cài đặt ứng dụng

Vào web store của chrome, tìm kiếm và cài đặt ứng dụng Advanced RESTclient cho trình duyệt.

<img src=http://i.imgur.com/OCPcBGi.png width="80%" height="80%" border="1">

Giao diện của ứng dụng: 

<img src=http://i.imgur.com/sPeLBDt.png width="80%" height="80%" border="1">

1. Địa chỉ url

2. Advanced RESTClient hỗ trợ các giao thức GET, POST, PUT, PATCH, DELETE, HEAD

3. Với các giao thức POST hoặc PUT ta cần thêm data khi gửi yêu cầu, ta thêm data ở ô số 3 này cũng dưới dạng raw hoặc form

4. Các option của ứng dụng

### Sử dụng Advanced RESTClient để test Openstack API.

####  Xác thực

Openstack gồm có Block Storage API, Indentify API ... bạn có thể tham khảo theo link sau: 

    http://developer.openstack.org/api-ref.html

Quá trình xác thực: Mỗi một yêu cầu mà gửi đến API đều yêu cầu có X-Auth-Token header. Các máy client sẽ chứa tokens này, sử dụng để gửi  đến các dịch vụ khác. Qúa trình xác thực được thể hiện ở đây: 

<img src=http://i.imgur.com/iUN45CW.png width="80%" height="80%" border="1">


####  Các bước tiến hành 

Trong phần này mình sẽ demo cách list ra các tenants trong Openstack

B1: Lấy tokens hệ thống

<img src=http://i.imgur.com/YZaPdrJ.png width="80%" height="80%" border="1">

1. URL gồm có địa chỉ của controller, port keystone service và API v2
2. Yêu cầu sử dụng giao thức POST
3. Chèn data gồm có username, password, tennantname
```sh

    {
    
        "auth": {
        
            "tenantName": "admin",
            
            "passwordCredentials": {
            
            "username": "admin",
            
            "password": "password123"
            
          }
        
          }
        } 

````

4. Thiết lập Set "Content-Type" header
5. Gửi yêu cầu

Phản hồi về 400 hoặc 401 HTTP có nghĩa là request sai URL hoặc data sai định dạng, phản hồi 200 HTTP là xác thực thành công và trả về file json chứa các thông tin các service của dịch vụ và token của user admin

Nếu thành công ta sẽ nhận được tokens của user như hình ảnh dưới đây: 

<img src=http://i.imgur.com/rNsQXkY.png width="80%" height="80%" border="1">

B2: List các tenant 

Để list các tenants thì tokens mà lấy từ bước phải là tokens của user có quyền admin. 

Xem mô tả lại API của list tenants

<img src=http://i.imgur.com/aybKfiN.png width="60%" height="60%" border="1">

Sử dụng Advaned REST Client lấy về danh sách các tenant:

<img src=http://i.imgur.com/1Qmx6E4.png width="80%" height="80%" border="1">

1. URL (ô số 1) gồm địa chỉ của controller, API v2.0/tenants
2. List tenant sử dụng phương thức GET
3. Thêm header cho yêu cầu với Key là X-Auth-Token giá trị tokens id lấy ở bước 1. 

Nếu gửi yêu cầu thành công sẽ nhận được phản hồi như sau: 

<img src=http://i.imgur.com/ghPZPWe.png width="60%" height="60%" border="1">

# Sử dụng python và REST client. 

Để sử dụng python gửi các yêu cầu HTTP ta sử dụng library request được viết bởi python 

Để sử dụng library request ta chỉ cần import thư viện này vào đầu đoạn script

    import request


Dưới đây là đoạn code mình viết bằng python để lấy tokens của openstack.

    import requests
    import json

    def gettoken():
        url ='http://172.16.69.70:35357/v2.0/tokens'
        data = {"auth": {"tenantName":"admin","passwordCredentials":{"username":"admin","password":"Welcome123"}}}
        a = requests.post(url,json.dumps(data),headers = {'Content-Type':'application/json'})
        if a.status_code !=200:
    	    raise Exception("Platform9 login returned %d, body: %s" %(a.status_code, a.text))
        else:
    	    response = a.json()
    	    return response

    respon = gettoken()
    token =  respon['access']['token']['id']
    printf token

Trong đoạn code này mình sẽ in ra màn hình token của user admin

Hàm thực hiện list các tenant:

    def listtenant():
        url ='http://172.16.69.70:35357/v2.0/tenants'
        respon = gettoken()
        token = respon['access']['token']['id']
        c = requests.get(url,headers = {'X-Auth-Token':b,'Content-Type':'application/json'})
        return c.json()['tenants']


# Cài đặt Trên Chrome
## Cài đặt ứng dụng
 Vào web store của chrome, tìm kiếm và cài ứng dụng Advanced RESTClient về cho trình duyệt
 
 
 <img src=http://i.imgur.com/SE9BloY.png width="90%" height="90%" border="1">

Giao diện của ứng dụng như dưới:

<img src=http://i.imgur.com/ACRfcit.png width="90%" height="90%" border="1">


1. Advanced RESTClient hỗ trợ các giao thức GET, POST, PUT, PATCH, DELETE, HEAD
2. Chèn thêm header cho yêu cầu, có thể chèn theo raw hoặc theo form
3. Với các giao thức POST hoặc PUT ta cần thêm data khi gửi yêu cầu, ta thêm data ở ô số 3 này cũng dưới dạng raw hoặc form
4. Các option của ứng dụng

## Sử dụng Advanced RESTClient để test OPENSTACK API

Openstack gồm có Block Storage API, Compute API, Data service API, Compute API, Indentify API, Image Service API, Networking API, Object Storage API,v.v.. Bạn có thể tham khảo tại:

```sh
http://developer.openstack.org/api-ref.html
```

Dưới đây mình sẽ thực hiện sử dụng ứng dụng Advanced REST Client giao tiếp với Indentify API. Cụ thể hơn là lấy tokens(POST request),hiển thị ra danh sách tenant(GET request),danh dách role(GET request)

###Bước 1: Xác thực hệ thống

Khi giao tiếp với bất kỳ thành phần nào trong OpenStack đều cần có xác thực. Để hiển thị lên danh dách các user, danh sách role, danh sách tenant cần có Token admin để xác thực. Để lấy được token này ta sử dụng giao thức POST với API v2.0. Xem mô tả API này tại:
```sh
http://developer.openstack.org/api-ref-identity-v2.html
```
<img src=http://i.imgur.com/ysVBi2V.png width="90%" height="90%" border="1">

Sử dụng Advanced REST Client lấy token:

<img src=http://i.imgur.com/7Dvl3Fs.png width="90%" height="90%" border="1">

1. URL( ô số 1) gồm có địa chỉ của controller, port keystone service và API version 2.0
2. Yêu cầu sử dụng giao thức POST( ô số 2)
3. Chèn data gồm có username, password, tenant name
```sh
{
    "auth": {
        "tenantName": "admin",
        "passwordCredentials": {
            "username": "admin",
            "password": "password123"
        }
    }
}
```
4. Thiết lập Set "Content-Type" header(ô số 4)
5. Gửi yêu cầu

Phản hồi về 400 hoặc 401 HTTP có nghĩa là request sai URL hoặc data sai định dạng, phản hồi 200 HTTP là xác thực thành công và trả về file json chứa các thông tin các service của dịch vụ và token của user admin

<img src=http://i.imgur.com/iyv7zjJ.png width="90%" height="90%" border="1">

Ta sẽ lấy token này để xác thực khi sử dụng các dịch vụ khác với các service khác trong OpenStack

###Bước 2: Tương tác với các Indentify API trong OpenStack

#### Show list tenant

Để xem danh sách tenant trong hệ thống OpenStack ta sử dụng Indenty API. Xem mô tả API:

<img src=http://i.imgur.com/VQdv5SA.png width="90%" height="90%" border="1">

Sử dụng Advaned REST Client lấy về danh sách tenant:

<img src=http://i.imgur.com/3ghkscX.png width="90%" height="90%" border="1">

1. URL (ô số 1) gồm địa chỉ của controller, API v2.0/tenants
2. GET request( ô số 2)
3. Thêm header cho yêu cầu với key là X-Auth-Token, giá trị là token id đã lấy ở bước trên

Gửi yêu cầu thành công hệ thống sẽ trả về file dưới định dạng json chứa danh sách các tenant
```sh
{
tenants_links: [0]
tenants: [4]
0:  {
description: null
enabled: true
id: "117d831515bb4d57aeb53977f25747d5"
name: "admin"
}-
1:  {
description: null
enabled: true
id: "21d5172005d24a3995394ab8b4b1d280"
name: "service"
}-
2:  {
description: null
enabled: true
id: "bf0a7712b2ce407eb904ddda7307730b"
name: "demo"
}-
3:  {
description: null
enabled: true
id: "ed4435642d664ed982760b195c1c647f"
name: "invisible_to_admin"
}-
-
}
```

#### Show list roles

Để xem danh sách role trong hệ thống OpenStack ta sử dụng Indenty API. Xem mô tả API:

<img src=http://i.imgur.com/fMC5yMg.png width="90%" height="90%" border="1">

Sử dụng Advaned REST Client lấy về danh sách roles:

<img src=http://i.imgur.com/jKtBurc.png width="90%" height="90%" border="1">

1. URL (ô số 1) gồm địa chỉ của controller, API v2.0/OS-KSADM/roles
2. GET request( ô số 2)
3. Thêm header cho yêu cầu với key là X-Auth-Token, giá trị là token id đã lấy ở bước trên

Gửi yêu cầu thành công hệ thống sẽ trả về file dưới định dạng json chứa danh sách các role
```sh
{
roles: [6]
0:  {
id: "46c58f6dc64f4395857b1c05c66abf9a"
name: "KeystoneServiceAdmin"
}-
1:  {
id: "73b91fa6f1a442818c077e992d1db8e3"
name: "KeystoneAdmin"
}-
2:  {
enabled: "True"
description: "Default role for project membership"
name: "_member_"
id: "9fe2ff9ee4384b1894a90878d3e92bab"
}-
3:  {
id: "bd8bae0ee9634823a3ea543c2672532f"
name: "Member"
}-
4:  {
id: "d2b112985ab24b29809e31ede55d1c31"
name: "ResellerAdmin"
}-
5:  {
id: "f0e77c2555ac4df2b276a2f560ae55a9"
name: "admin"
}-
-
}
```


