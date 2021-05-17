# How to build a web server on GCP
###### tags: `gcp`, `flask`, `python`, `web`, `linux`
- 先在 GCP 創建一個 VM instance
- 接著嘗試透過 ssh 去跟 VM 連線
- 在 VM 建置 python 環境
- 設定並啟動flask
- 讓 flask process 在背景中運行(從背景中重起 flask process )

## 創建一個 VM instance
先到 compute engine 新增一個 VM instance，要記得勾選 **Allow HTTP traffic**，**Allow HTTPS traffic**，然後要設定 external IP。

## 透過 ssh 跟 VM 連線
1. 首先我們必須先產生 ssh public key 跟 private key：
`ssh-keygen -t rsa -f ~/.ssh/[KEY_FILENAME] -C [USERNAME]`

2. 如果 key 產生成功，key 會出現在以下位置：
    Public key file: `~/.ssh/[KEY_FILENAME].pub`
    Private key file:`~/.ssh/[KEY_FILENAME]`

3. 接著到 compute engine 的 metadata 頁面新增 public key，將 public key 的內容複製過來。

4. 新增成功過後，使用 ssh 指令跟 VM 連線：
`ssh -i ~/.ssh/my-ssh-key [USERNAME]@[IP_ADDRESS]`
在本範例中，private key 位於 `~/.ssh/my-ssh-key`。

## 在 VM 建置 python 環境
1. 安裝 python：
    ```
    $ sudo apt update
    $ sudo apt install python3
    ```
2. 建置 python virtual environment 並啟動：
    ```
    $ python3 -m venv [DIR_PATH]
    $ source [DIR_PATH]/bin/activate
    ```
3. 利用 requirements file 安裝專案所需的 python 套件：
    ```
    $ pip install -r [REQUIREMENTS_FILE_NAME]
    ```
    如何產生 requirements file，可以透過 freeze 指令：
    ```
    $ pip freeze 
    $ pip freeze > [REQUIREMENTS_FILE_NAME]
    ```

## 設定並啟動flask
1. 在專案目錄底下做相關設定，並啟動：
    ```
    $ export FLASK_APP=[RUN_FILE_NAME]
    $ flask run --host=0.0.0.0
    ```    
    若將 IP 設為 0.0.0.0 表示對外開放所有連線，需要切換成 root 權限：
    ```
    $ sudo su
    ```

## 讓 flask process 在背景中運行
1. 使用指令 `&` 加在程式名後面，就可將在背景運行，例如：
    ```
    $ flask run &
    ``` 
2. 使用`top`程式或是`pgrep`得知 process 的 process id (PID)，可使用`fg`程式將 process attach 回來，或是使用`kill`程式將 process 關閉。
    ```
    $ top | grep [PROCESS_NAME]
    $ pgrep [PROCESS_NAME]
    $ fg [PROCESS_ID]
    $ kill [PROCESS_ID]
    ```
> 提醒！！目前只能使用 http 連線而不是 https。