---
layout: post
title: capstone-project
author: [LIN JIAN-CHEN,ALAN LEE]
category: [Lecture]
tags: [jekyll, ai]
---
## 期末專題 - 使用 YOLOv5 辨識水下魚種 

**專題實作步驟:**
1. 建立魚類標註資料集 <br>
2. 挑選目標魚種<br>
3. 使用高斯雜訊對影像進行擴增<br>
4. 使用顏色通道調整對影像進行擴增<br>
5. 使用Yolov5m模型搭配 yolov5.pt 權重檔進行遷移式學習訓練模型 (參數使用 Img size 1280 , batch size 16 ,epoch 200) <br>
56. 輸入影像進行辨識 <br>

* **模型建構** (PC with Anaconda & TWCC雲運算中心)<br>
* **程式樣本** (自行撰寫與參考Open source網站)

---

## YOLO系列介紹
**YOLO (You Only Look Once) 是一個 one-stage 的 object detection 演算法** <br> <br>
* **One stage** (速度較快,較多人研發在行動裝置上) <br> 
物件位置偵測和物件辨識一步到位，也就是一個神經網路能同時偵測物件位置也可以辨識物件 <br>
著名:YOLO系列,Single Shot Detector (SSD) <br>
![image](https://user-images.githubusercontent.com/95911604/206740169-dcfff867-c950-4ddb-b244-474b5db16001.png)

* **Two stage** (辨識精確度較高,辨識時間較長) <br> <br> 
先用特殊方法先選出Region of Interesting，然後針對選出的物件(Region Proposals)再進行物件辨識 <br> <br>
著名:Fast-RCNN <br>
![image](https://user-images.githubusercontent.com/95911604/206740196-5e7b5f6c-7162-495c-bb50-62b64ab9bdb5.png) <br><br>

## 基本名詞介紹

* **Precision (準確率)** : 在所有預測為正樣本中，有多少為正樣本 TP / (TP + FP) <br> <br>

* **Recall (召回率)** : 在所有正樣本當中，能夠預測多少正樣本的比例 TP / (TP + FN) <br> <br>

* **Precision高的模型 = 較謹慎 (雖常沒辦法抓出命名實體，但只要有抓出幾乎都正確)** <br> <br>

* **Recall   高的模型 = 較寬鬆 (雖然有時候會抓錯，但幾乎該抓的都有抓到)** <br> <br>

* **AP** : PR curve (Precision-Recall curve) 的面積 (area under curve, AUC) <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206674998-3e682d09-748d-4ccf-97b5-d2ecb0c9c19a.png)

* **mAP** :每一種物體的AP算完後的平均值 <br> <br>

* **評估指標 IOU (Intersection over Union)** - 兩個 bndBox 的交集 / 兩個 bndBox 的聯集 <br><br>
IoU > 閥值(Threshold) = TP <br>
IoU < 閥值(Threshold) = FP <br>
![image](https://user-images.githubusercontent.com/95911604/206674353-8636ddc2-edcf-4fce-b7b3-53f0df008133.png)

---

* **Confusion Matrix 混淆矩陣** <br><br>
第一個英文字:預測正確(T)或錯誤(F) <br>
第二個英文字:預測是目標物件(P)OR不是目標物件(N) <br>
![image](https://user-images.githubusercontent.com/95911604/206675135-764eb193-1993-4cd7-9ff5-4244749db716.png)

---
## 標註照片

* **YOLO 的 Label 格式** <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206723888-88603963-1ded-42c2-8a44-0c47d6c1694c.png) <br>

![image](https://user-images.githubusercontent.com/95911604/206724587-8b7cd47f-0760-4eb2-b61d-2de1eb811ed0.png) <br>

* **使用LableIMG進行影像標註** <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206864022-060f451e-3f0d-470d-b7fe-a824c2070cdf.png) <br>

---

## YOLOv5實際操作

**環境建構** <br>
* **使用anaconda** <br><br>
![image](https://user-images.githubusercontent.com/95911604/206863407-395d3083-5ecf-40c4-af8a-aeafb03d87c5.png) <br><br>
Conda 是一個開源的跨平台工具軟體 <br>
傳統 Python 使用者以 pip 作為套件管理員（package manager）、以 venv 作為工作環境管理員（environment manager），而 conda 則達成了「兩個願望、一次滿足」既可以管理套件亦能夠管理工作環境。

<br>


**安裝與建構環境** <br><br>
這個小節使用的常見 conda 指令有：

- conda --version 檢視 conda 版本
- `conda update PACKAGE_NAME`更新指定套件
- conda --help 檢視 conda 指令說明文件
- conda list --ENVIRONMENT 檢視指定工作環境安裝的套件清單
- conda install PACAKGE_NAME=MAJOR.MINOR.PATCH 在目前的工作環境安裝指定套件
- conda remove PACKAGE_NAME 在目前的工作環境移除指定套件
- conda create --name ENVIRONMENT python=MAIN.MINOR.PATCH 建立新的工作環境且安裝指定 Python 版本
- conda activate ENVIRONMENT 切換至指定工作環境
- conda deactivate 回到 base 工作環境
- conda env export --name ENVIRONMENT --file ENVIRONMENT.yml 將指定工作環境之設定匯出為 .yml 檔藉此複製且重現工作環境
- conda remove --name ENVIRONMENT --all 移除指定工作環境

創建新環境

- conda create -n your_env_name python=x.x
- activate your_env_name 激活環境
- conda activate your_env_name 前往環境

刪除環境

- conda remove -n your_env_name --all
- conda remove --name your_env_name package_name 刪除環境中的某個包

看環境有哪些包

- conda list

看環境有哪些環境變數

- conda env list

---

## Dataset 準備與架構 

**訓練用數據集 (From 李東霖老師-智慧生活實驗室-海科館水下攝像鏡頭)** <br>

* **資料挑選 (由於魚種數量分布不均,使用黑白名單方法挑選)** <br>
 ![image](https://user-images.githubusercontent.com/95911604/206824916-5b0f6d8d-8698-46be-a888-663f1e3cbf0f.png) <br>
![image](https://user-images.githubusercontent.com/95911604/206824948-9c9b3f01-8112-4152-818b-e9e6f929339c.png) <br>

* **魚種編號** <br>
![image](https://user-images.githubusercontent.com/95911604/206745881-30858cff-2c05-4f23-b19f-5ce4315e0376.png) <br><br>


**1. 挑選平均過後的影像以 80:20 進行分割 (3940張Train 1053張val)** <br>

![image](https://user-images.githubusercontent.com/95911604/206747666-b1b8703c-b241-4dc9-891f-d34ef3178332.png) <br> <br>

**2. 新增Noise (高斯雜訊) 進行資料擴充 (7880張train 2106張val)** <br>
![image](https://user-images.githubusercontent.com/95911604/206748539-145579f7-9a54-4d30-a5f7-56b241f15cb6.png) <br>

![image](https://user-images.githubusercontent.com/95911604/206748640-e99c1662-3aef-4eba-afcc-7c7c45c1bdf3.png) <br>

![image](https://user-images.githubusercontent.com/95911604/206748715-f91efa0f-57cf-4ab1-a8be-bab6aaa96bcc.png) <br> <br>

**3. 調整顏色通道進行資料擴充** <br><br>

**先對影像顏色擴增進行實驗 (315張Training,針對1,3編號進行擴增) 觀察如何調整可以提升PR與mAP值** <br>

* **訓練參數** <br> <br>
1.只挑選有編號1,3的照片(315張) + 依各顏色擴增方法擴增(315張) 為訓練集 <br>
2.2.1015張照片為驗證集 <br>
3.參數 python train.py --img 640 --epoch 150 --batch-size 8 --data data/lundata.yaml --cfg cfg/yolov5s.yaml --weight weights/yolov5s.pt <br>

![image](https://user-images.githubusercontent.com/95911604/209088736-38a95194-f1d4-4851-b076-8ddc16e53f79.png) <br>
![未命名1](https://user-images.githubusercontent.com/95911604/209089073-00d7a382-f2df-4378-88f7-e8f27add7414.png) <br>

 * **總結:挑選綠色進行擴增較有效果** <br> 
![image](https://user-images.githubusercontent.com/95911604/209090534-e7e91286-0f15-47a6-803d-c485bff90076.png) 


---

## 最終訓練參數與結果

* Yolov5自帶4種不同大小的模型提供訓練的選擇 <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206824780-cb27d94a-42b9-4ea6-a6c2-cdbd184d3213.png) <br>

* 採用YOLOv5m模型（Img size 1280 , batch size 16 ,epoch 200） <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206749074-db12172e-9479-473d-8124-917cdca517ef.png) <br>

**訓練設定** <br>

* **更改.yaml檔裡的nc數及標籤** <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206740736-a8a467da-9bd1-426f-bcaf-717ad9cc78df.png) <br>

* **更改cfg檔nc數** <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206741855-3f46ffc5-cc9d-45ac-bb0d-3eb80feaacde.png) <br>

* **執行訓練指令 (範例)** <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206744712-e7115403-59fa-4a2b-99c9-1f614ab9ab60.png)

* **混淆矩陣** <br>
![image](https://user-images.githubusercontent.com/95911604/206749373-ac28759c-3987-4930-af70-f4c1f2018911.png) <br> <br>

* **應用模型偵測影像檔** <br> <br>
![image](https://user-images.githubusercontent.com/95911604/206825153-ae3ab7fb-306f-4ddc-b6a8-bc2065145360.png) <br> <br>


<iframe width="723" height="482" src="https://www.youtube.com/embed/46wfrbQC8fI" title="影像偵測1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


---

## 小組成員
1. **電機3B 00953117 李宇倫** : 標註資料,資料擴增,建構環境,篩選數據,訓練模型並應用,整理成果檔 <br>
2. **電機3A 00953034 林建辰** : 標註資料,資料收集,網頁撰寫及編排,模型參數調整
---

<br>

This site was last updated {{ site.time | date: "%B %d, %Y" }}.
