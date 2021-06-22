# Docker

## 簡介

Docker 是一種輕量級虛擬化的技術，可快速搭建虛擬環境，不適合集合大量工作在一個 Docker 上，這樣維護起來才會比較輕鬆。

Docker 須持續執行任務，否則執行完便會死亡。

## 常用指令

> p.s. ID 只需輸入可與其他鏡像/ docker 區隔的部分即可

* `docker images`：查詢鏡像

* `docker ps`：列出正在執行的 docker

  > -a：列出所有 docker ，包含正在執行和已經死亡的
  >
  > -q：列出 docker 的 ID

* `docker rmi ImageID`：刪除鏡像

* `docker rmi Image:Tag`：刪除鏡像

* `docker pull Image:Tag`：拉取鏡像（Tag 可不寫，預設為 Latest）

* `docker run -it ubuntu echo "Hi"`：執行 ubuntu 鏡像，顯示 Hi 後，因無其他工作，所以此 docker 死亡

* `docker run -itd -p 8080:80 httpd`：執行 httpd docker，綁定本機 8080 埠

  > -i，--interactive：讓 docker 的標準輸入保持打開
  >
  > -t，--tty：讓 Docker 分配一個虛擬終端（pseudo-tty）並綁定到 docker 的標準輸入上
  >
  > -d，--detach：讓 docker 處於背景執行狀態並印出 dockerID
  >
  > --name：指定 docker 名稱
  >
  > -p，--publish：將 docker 發布到指定的埠號
  >
  > --rm： docker 工作完自動刪除

* `docker rm dockerName`：刪除狀態是 Exited 的 docker

* `docker rm dockerID`：刪除狀態是 Exited 的 docker

* `docker rm -f dockerID`：強制刪除執行中的 docker
  * 可結合 `docker ps -a -q` 強制刪除所有的 docker
  * 完整指令：<code>docker rm -f \`docker ps -a -q\`</code>，每次回傳一個 dockerID 去刪除

* `docker stop dockerID`：關閉執行中的 docker（docker Up -> Exited）

* `docker start dockerID`：開啟已關閉的 docker（docker Exited -> Up）

* `docker exec -it dockerID`：進入 docker  docker 內

* `docker commit dockerID ImageName:Tag`：以 docker 現在的樣子建立鏡像

* `docker tag ImageName:Tag (New/Old)ImageName:(New/Old)Tag`：重新建立鏡像標籤

* `docker login`：登入 Docker Hub

* `docker push ImageName:Tag`：上傳 Docker 鏡像

  * 鏡像名稱規範：Server/Account/Name:Tag，若 Server 為 Docker 時可省略

## Dockerfile

### 簡介

自動產生鏡像用的腳本

### 範例

* Dockerfile

      FROM centos:centos7 // 新鏡像源自 centos:centos7
      RUN yun -y install httpd // 自動安裝 httpd
      EXPOSE 80 // 開放 80 埠
      ADD index.html /var/www/html // 複製本地 index.html 到 docker /var/www/html

* index.html

      hi

* 建置指令

      # docker build -t centos:web1.0

* 執行指令（讓網頁伺服器跑在前景）

      # docker run -d -p 8080:80 centos:web1.0 /usr/sbin/apachect1 -DFOREGROUND

---

## 參考資料

1. iT邦幫忙：https://ithelp.ithome.com.tw/articles/10193534