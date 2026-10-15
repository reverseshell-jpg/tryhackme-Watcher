# tryhackme-Watcher
1.RECON
Server mở 3 cổng gồm 21-ftp, 22-ssh, 80-http
<img width="684" height="188" alt="image" src="https://github.com/user-attachments/assets/63e7af1a-736f-424c-a59d-cc37d1c73684" />
Sử dụng dirsearch để tấn công fuzzing directory
<img width="562" height="678" alt="image" src="https://github.com/user-attachments/assets/542bdf08-4e1a-42e5-87d5-70d147997ed3" />
Nội dung của `robots.txt`:
<img width="336" height="88" alt="image" src="https://github.com/user-attachments/assets/22fce656-924c-4c96-83c8-c90778de548c" />
Tôi đã thử truy cập vào endpoint `/secret_file_do_not_read.txt` tuy nhiên kết quả nhận về là 403 forbidden
Quay lại trang chủ của website, quan sát và tôi nhận ra website dính LFI-(https://www.indusface.com/learning/file-inclusion-attacks-lfi-rfi/)
<img width="355" height="38" alt="image" src="https://github.com/user-attachments/assets/d3b60aec-46a7-4e0e-badf-f1edb80266c5" />
<img width="1267" height="384" alt="image" src="https://github.com/user-attachments/assets/82915d1d-aee6-4267-83e3-6b65d69769f9" />
Tôi lại thử đọc file 'secret_file_do_not_read.txt' thông qua LFI:
<img width="857" height="48" alt="image" src="https://github.com/user-attachments/assets/7c8e1760-24ee-4447-bf5b-90001310ce6f" />
Từ nội dung của đoạn text ta biết được tài khoản của ftp và user này có thể ghi và lưu tệp vào /home/ftpuser/ftp/files, chúng ta có thể lợi dụng điều này để upload reverse shell và kích hoạt nó thông qua LFI
2. EXPLOIT
2.1 Read flag_1:
<img width="2169" height="725" alt="image" src="https://github.com/user-attachments/assets/88051fc4-bca8-4d64-8027-cd3597e09c05" />
2.2 Read flag_2:
Đăng nhập vào ftp bằng tài khoản có được ở bên trên và đọc flag_2.txt:
<img width="2167" height="726" alt="image" src="https://github.com/user-attachments/assets/20d54e3d-fb22-4a1d-b33f-aff91fb3352d" />
2.3 Read flag_3:
Truy cập vào thư mục files và dùng lệnh put để upload reverse shell
<img width="387" height="112" alt="image" src="https://github.com/user-attachments/assets/afa2c2f8-2294-4335-bd65-72923c6c05a6" />
Sử dụng LFI để kích hoạt payload
<img width="1106" height="210" alt="image" src="https://github.com/user-attachments/assets/2438d987-11bf-48a1-9d51-5254906a5550" />
Dùng find để tìm `flag_3.txt ` và đọc flag
<img width="2170" height="725" alt="image" src="https://github.com/user-attachments/assets/db9db0dc-6a13-4bc1-a763-b6ddd41e51f7" />
2.4 Read flag_4:
Sử dụng `sudo -l` để kiểm tra xem có gì đặc biệt:
<img width="776" height="116" alt="image" src="https://github.com/user-attachments/assets/0bee9174-fc66-438b-b0ac-74401ea27dca" />
Từ đây có thể thấy chúng ta có thể thao tác với hệ thống bằng user toby
Đọc flag
<img width="531" height="36" alt="image" src="https://github.com/user-attachments/assets/13bda706-dc3d-4540-a700-d0e4d4ec9fe4" />
2.5 Read flag_5
Kiểm tra file /etc/crontab:
<img width="379" height="33" alt="image" src="https://github.com/user-attachments/assets/c27c73d0-e746-4a89-9237-0bf830885270" />
<img width="578" height="33" alt="image" src="https://github.com/user-attachments/assets/f7ed4a4a-b8ff-4bda-a0a3-ab1db9e17d99" />
Vậy ta có thể lợi dụng user toby để sửa lại tệp cow.sh và lấy shell mới với tài khoản của mat
<img width="645" height="120" alt="image" src="https://github.com/user-attachments/assets/e5006615-3592-4756-84b9-480e18d8bde2" />
Đọc flag
<img width="316" height="34" alt="image" src="https://github.com/user-attachments/assets/91b57f24-6e40-4373-a2d6-3f3de46aa4bd" />
2.6 Read flag_6
<img width="438" height="395" alt="image" src="https://github.com/user-attachments/assets/6a11bd04-8cdd-4bd8-8d4e-1ebef6ff32ad" />
<img width="777" height="115" alt="image" src="https://github.com/user-attachments/assets/51f6b65d-8ef9-4536-8aaa-7a2959630486" />
Giống như lúc trước, chúng ta có thể thực thi file /home/mat/scripts/will_script.py bằng user của will
Có thể thấy file will_scripts.py import file cmd.py và sau khi kiểm tra có thể thấy user mat có thể sửa file cmd.py
Chúng ta có thể lợi dụng điều này để tiêm payload vào file cmd.py, khi file will_scripts.py import file và thực thi thì ta sẽ có được shell với user will
<img width="781" height="27" alt="image" src="https://github.com/user-attachments/assets/9c8b1d41-3fd7-44fe-8be2-d5d041e6458d" />
<img width="558" height="126" alt="image" src="https://github.com/user-attachments/assets/6b09bdec-44e1-4a93-9e0d-9b982fe0d311" />
Đọc flag
<img width="345" height="33" alt="image" src="https://github.com/user-attachments/assets/fa2b642c-cf97-4ae1-9749-889346e89247" />
2.7 Read flag_7







