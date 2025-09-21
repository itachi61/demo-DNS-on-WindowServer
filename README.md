# demo-DNS-on-WindowServer

Demo DNS Server on Window Server 12
1. Cài đặt cấu hình mạng cho máy.
<img width="802" height="600" alt="Screenshot 2025-09-20 083934" src="https://github.com/user-attachments/assets/046e3c1a-97f3-424d-96eb-02e95d854e67" />

+ Chọn Ethernet0 ( góc phải trên màn hình )
+ sau đó chọn Properties
+ Tắt Internet Protocol Version 6
+ Chọn Internet Protocol Version 4 --> chọn Properties
sau đó cấu hình như ảnh 
<img width="415" height="454" alt="image" src="https://github.com/user-attachments/assets/c00e40e3-d719-4b15-a03a-e8199a605f50" />
( ở máy 2 làm tương tự nhưng IP là 192.168.1.10 )

2. Tải DNS về cho máy

+ Vào Manage -> chọn Add Roles and Features 
+ bấm Next đến Option Server Zone -> chọn DNS sau đó lại next đến khi click được install và đợi download hoàn tất.

3. Cấu hình cho DNS
3.1 
+ Vào Tool chọn DNS sau đó chọn Forward Lookup Zones -> tạo new zone ( Primary zone )
+ Đặt tên zone name là sgu1.com ( có thể đặt tên tùy ý )
+ Chọn Allow both để phục vụ cho việc Forwarder and backup dễ hơn
+ sau đó finish

3.2 
+ Vào Reverse Lookup Zones để tạo 1 phân giải ngược. 
+ với IP 192.168.1
+ Chọn Allow both để phục vụ cho việc Forwarder and backup dễ hơn

3.3
+ Trong sgu1.com tạo ra 1 Host với tên là www với IP là 192.168.1.1
+ tạo thêm 1 Host có tên mail với IP là 192.168.1.5
+ tạo thêm 1 Alias(CNAME) sau đó browse vào địa chỉ của www.sgu1.com
+ tạo thêm 1 Mail Exchanger(MX) sau đó browse vào địa chỉ của mail.sgu1.com
+ vào Reverse Lookup Zones tạo 1 PTR
<img width="777" height="535" alt="image" src="https://github.com/user-attachments/assets/5e96f554-5c67-404b-bcc5-1e00dc7b56b8" />

4. kiểm tra 
+ vào PowerShell ping đến ip máy 192.168.1.1 để nhận được phản hồi
+ dùng lệnh "nslookup" để kiểm tra ip và tên miền của DNS
<img width="879" height="642" alt="image" src="https://github.com/user-attachments/assets/48bc398b-4818-470d-894c-d8fdc7b6eef8" />

** làm tương tự cho máy 2 ở bước 1 và 2 (lưu ý địa chỉ là 192.168.1.10) "

<img width="1214" height="877" alt="image" src="https://github.com/user-attachments/assets/2c0df52c-86d8-48f5-9a2a-a043e8a01fc2" />

5. Cấu hình DNS cho máy 2
+ Trong sgu2.com tạo ra 1 Host với tên là www với IP là 192.168.1.10
+ Vào Reverse Lookup Zones để tạo 1 phân giải ngược. 
+ vào Reverse Lookup Zones tạo 1 PTR
+ tạo thêm 1 Alias(CNAME) tên sgu sau đó browse vào địa chỉ của www.sgu2.com

<img width="669" height="413" alt="image" src="https://github.com/user-attachments/assets/9a5194ef-738a-464d-b609-dafd4a152123" />

6. Thực hiện Forwarder giữa 2 máy.
+ Tắt tưởng lửa ( firewall ) ở 2 máy (Ở Local Server có dòng Window Firewall, nhấp vào dòng chữ kế bên nó "Public: On" -> vào Turn Window Firewall on or off -> Chọn cả 2 mục "turn off"
+ Click chuột phải vào file sgu1.com ở máy 1 và vào Zone Transfer sau đó thêm địa chỉ của máy 2 để phục vụ cho việc forwarder (192.168.1.10), dùng edit để thêm vào sau đó apply.
<img width="1214" height="666" alt="image" src="https://github.com/user-attachments/assets/a9692991-5c52-4d90-8f06-9e3f2dc8cac4" />

**làm tương tự với máy thứ 2**

Kiểm thử:
Vào powershell máy 1 ping đến IP Dns máy 2 và tên miền máy 2 và ngược lại
<img width="530" height="399" alt="image" src="https://github.com/user-attachments/assets/cb8e732c-ff1a-4827-b227-3c0381959e1c" />

7. Thực hiện Backup giữa 2 máy

Ở đây nhóm mình thực hiện tạo 1 Zone ( secondary zone ) ở máy 2 để backup cho máy 1
Việc backup này sẽ giúp DNS chúng ta vẫn hoạt động được Khi DNS ở máy 1 bị hư hoặc die DNS.

+ Mở DNS Manager → chuột phải vào zone (vd: sgu1.com) → Properties.
+ Vào tab Zone Transfers → tick Allow zone transfers.
+ Chọn: Only to the following servers → thêm IP 192.168.1.10.

+ Vào Forward Lookup Zone ở máy 2 , tạo thêm 1 Secondary zone có tên sgu1.com và địa chỉ 192.168.1.1
+ Finish → DNS2 sẽ tự động tải zone từ DNS1.

Hoặc nếu như làm như video chúng mình demo (Cách ở trên đơn giản hơn là chỉ thẳng vào máy cần backup, cách ở trong video là định nghĩa từng máy để 2 máy hiểu máy nào là máy cần backup và máy backup.

+ Mở DNS Manager → chuột phải vào zone (vd: sgu1.com) → Properties.
+ Vào tab Zone Transfers → tick Allow zone transfers.
+ Chọn: Only to servers listed oin the Name Servers tab

+ Vào Name Servers, nhấn vào Add
+ Thêm từng Fully qualified domain name (FQDN) tương ứng với IP Addresses:
  (www.sgu1.com - 192.168.1.1) (sgu2.sgu1.com - 192.168.1.10)
+ Nhấp vào Apply và nhấp OK

+ Ở Reversed Lookup Zone cũng làm giống y chang các bước trên
+ Refresh cả 2 zone.
Để kiểm thử thành công hay chưa ta có thể tạo thêm 1 Host ở máy 1 sau đó xem nó có tự động thêm vào ở máy 2 hay không.

+Máy 1:
<img width="825" height="618" alt="image" src="https://github.com/user-attachments/assets/91b4f12e-9a4e-41df-9248-53e6c269a988" />
+Máy 2: 
<img width="1214" height="877" alt="image" src="https://github.com/user-attachments/assets/7a88635e-daf3-4443-97ed-0db8b7d291f1" />

Vậy là đã thành công việc Backup dữ liệu.

Hoặc bạn có thể dùng lệnh ở PowerShell
Bạn có thể dùng lệnh:

"nslookup www.dns1.com 192.168.1.10"


Nếu phân giải được là thành công.






