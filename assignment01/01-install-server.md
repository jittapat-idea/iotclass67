# Install Server and Docker
  กลุ่มของเราได้เลือกใช้ Server เป็น Ubuntu Server เป็นเครื่อง Server ของเรา เมื่อได้ทำการลง
  OS เสร็จเรียบร้อย ก็จะทำการลง Docker ผ่าน Terminal แต่ก็จะต้องแก้ไขปัญหาเรื่อง Network ของเครื่องก่อน
  โดย กลุ่มเราได้แก้ปัญหาโดยใช้สาย LAN เชื่อมต่อเครื่องคอมพิวเตอร์ของเราโดยให้เครื่องคอมพิวเตอร์ เชื่อมต่อ
  Internet เอาไว้แล้วทำการ Bridge Network เพื่อให้เครื่อง Server อยู่ในวง LAN เดียวกัน

## How to install Server
  * Format flash drive ที่ไม่ได้ใช้งานอะไร หรือ ไม่มีข้อมูล เพื่อ Format image ของ OS Ubuntu Server
  * ทำการ Restart เครื่องแล้วเข้า Bios เพื่อที่จะเลือก Flash Drive ที่ไว้สำหรับลง Server
  * ทำการ เลือกภาษา, Time zone และ กรอก Username, Password
  * จากนั้นทำการลงเครื่องมือสำหรับการเช็ค ip network (net-tools) เพื่อให้คอมพิวเตอร์ส่วนตัว SSH เข้ามาได้ เพื่อง่ายต่อการทำงาน


## How to install Docker
> ขั้นตอนการตั้งค่า Docker repository เพื่อที่จะติดตั้ง Docker Engin ใน เครื่อง Host ใหม่
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
# Install the Docker packages:
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
# Verify that the Docker Engine installation is successful by running the hello-world image
sudo docker run hello-world
```

## Manage Docker as a non-root user
> ขั้นตอนการตั้งค่า Unix group เพื่อให้สามารถรันคำสั่งใน docker โดยไม่ต้องนำหน้าด้วย "sudo"

```bash
#Create the docker group.
sudo groupadd docker
```
```bash
#Add your user to the docker group.
sudo usermod -aG docker $USER
```
```bash
#run the following command to activate the changes to groups:
newgrp docker
```
```bash
#Verify that you can run docker commands without sudo.
docker run hello-world
```

