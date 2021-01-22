# 手臂程式燒錄
PS:我xyzrobot的程式需要用arduino-1.0.6的版本才能下載板子
- 先使用機械手臂/RCK100_6DOF資料夾裡的ino燒錄給xyzrobot，接著就可以使用xyzrobot的程式去寫動作
![](https://i.imgur.com/ouYy2Ie.png)
- 調整好動作後，把手臂位置記起來，接著去projectjustreadcomputer.ino可以用序列埠打f去測試動作
- 確定好連續動沒問題，可以去projectjustread把數據填入，接著串接上ros就可以用ros_topic控制手臂

PS:每個馬達有控制範圍限制
- ID1:0~1023
- ID2:234~742
- ID3:252~790
- ID4:152~782
- ID5:128~896
- ID6:512~852


# Line BOT
- 建立帳號
透過參考文章的方式建立一個LINEBOT帳號
[Linebot建立參考文章](https://blackmaple.me/line-bot-tutorial/)
- 帳號建立完成後，上傳圖文表單
打開line-bot-sdk-python-master/linebot.py
透過LINE Developers可以查看Channel access token
將Channel access token貼於*****上
1.新增圖文表單，將圖片及啟用的程式碼段落作註解然後執行，此時可以在視窗中得到richmenu-...
2.將richmenu-...貼於標示。。。的段落上
3.將新增註解(避免重複新增)，移除圖片註解，並執行
4.將圖片註解，並移除啟用的註解，執行後，若出現{}，代表上傳成功，可以透過LINEBOT做查看
[圖文表單參考文章](https://medium.com/enjoy-life-enjoy-coding/使用-python-為-line-bot-建立獨一無二的圖文選單-rich-menus-7a5f7f40bd1)
- 資料庫firebase建立
建立cloud firestore，並將資料庫建立如下圖所示
![](https://i.imgur.com/a2V0lBD.png)
打開專案/設定/服務帳戶/firebase admin sdk，點選產生新的私密金鑰，並將下載檔案放置於line-bot-tutorial-master當中，檔案當中即為js金鑰
[firebase資料庫參考文章](https://medium.com/@resnick1223)
打開專案中的hosting，並建立網址,透過visual studio code打開hosting資料夾
在執行處輸入firebase init hosting，登入firebase
修改config.js的檔案，改成自己的資料庫資料
執行firebase deploy上傳
[firebase網頁建立參考文章](https://william-weng.github.io/2019/11/30/firebase-hosting/)
- 程式上傳
將line-bot-tutorial-master/app.py中的程式覆蓋至第一步驟中所下載的資料夾中的app.py中，記得修改channel acess token,
channel secret,user_id(都可以透過line development找到)，js金鑰及網頁網址
requirements.txt檔也要替換過去
上傳程式照著網站  一樣是三步驟
1. git add .
2. git commit -m "Add code"
3. git push -f heroku master



# 建立地圖
- roslaunch rplidar_ros view_slam.launch
- rosrun rosaria RosAria
- rosrun rosaria_client teleop (用別台電腦打)操控車子
- rviz 開啟tf
- rosrun map_server map_saver -f 名稱

# 更新地圖到程式上
- 把地圖.yaml跟地圖.pgm丟到 src/p3dx_navigation/maps裡面
- 修改src/p3dx_navigation/launch/move_base_rosaria.launch   把地圖名稱換成自己的

# 操作步驟
- 接線
	- 鏡頭接樹莓派的USB  usb to ttl接樹莓派的GPIO
	- 示意圖
	  ![](https://i.imgur.com/24QSb7W.jpg)


- USB修改
	- 分別插上pioneer 3-DX、RP Lidar、usb to ttl、手臂
	- 使用ls /dev/tty* 這個指令可以查看USB port  可以插一個就打一次  得知每一個的port
	- pioneer 3DX
		- 修改src/rosaria/RosAria.cpp 292行  的USBport
		- 修改src/p3dx_navigation/launch/nav_pioneer.launch 第4行 的usb port
	- RP Lidar
		- 修改 src/rplidar_ros/launch 第3行 的USB port 
		- 修改src/p3dx_navigation/launch/nav_pioneer.launch 第13行 的usb port
	- usb to ttl
		- 修改publisher .py 的第13行 的usbport
	- 手臂
		- 在執行程式的時候 後面自己改

- 事先要開的東西
  - 車子上的電腦：
1.roslaunch p3dx_navigation move_base_rosaria.launch  打開move_base
2.roslaunsh p3dx_navigation nav_pioneer.launch  打開navigation
3.python publisher.py  接收由樹莓派用usb_to_ttl 傳來的資料
4.rosrun rosserial_python serial_node1.py /dev/ttyUSB2 與機械手臂進行通訊(USB幾請自己看機械手臂是連到哪一個port)

  - 操控者的電腦：
1.rosrun p3dx_navigation pioneer_deliver_lab506
2.rosrun beginner_tutorials pioneer_listener
3.rviz 切換到建立好的/map當中
4.使用2D Pose Estimate定位到機器人相對位置
5.rosservice call /request_nomotion_update 使紅點收斂後即可用手機下達指令


- 實際操作動作(送藥)
	- Line:按下開始送藥>病房1(Line會傳MQTT到車上電腦，車上電腦會利用剛剛開的publisher接收並轉換成ros_topic傳出去，這時操控者電腦接收到ros_topic封包後就會送出goal的訊號，車子就會開始導航到目的地。)
	- 到目的地後，會接著由操控者電腦送ros_topic到手臂，手臂會開始夾取杯子
	- 夾取完杯子後，操控者電腦會直接下達goal指令回家，並且傳送ros_topic給publisher.py，然後就會MQTT傳給line，Line接收到後會傳送給使用者，送藥完畢
- 實際操作動作(導航)
	- Line:按下開始導航(Line會傳MQTT到車上電腦，車上電腦會利用剛剛開的publisher接收並轉換成ros_topic傳出去，這時操控者電腦接收到ros_topic封包後就會送出goal的訊號，車子就會開始導航到目的地。)
	- 每到一個目標點，就會下達下一個goal
	- 到最後一個goal結束後，會傳送ros_topic給publisher.py，然後就會MQTT傳給line，Line接收到後會傳送給使用者，巡邏完畢
