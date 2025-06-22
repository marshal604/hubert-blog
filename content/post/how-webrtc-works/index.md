+++
title = 'WebRTC 運作原理筆記'
date = 2025-06-22T14:00:00+08:00
draft = false
featured_image = 'featured_image.png'
tags = ['Frontend', 'WebRTC']
+++

## Overview

工作上開始接觸 RTC SDK 相關的開發，想起過去有分享過 WebRTC 的運作原理，趁著週末有空檔來重新複習一下。

## 什麼是 WebRTC

WebRTC 全名是 Web Real-Time Communication，是一個開放標準，用於在瀏覽器中實現實時音訊、視訊和數據傳輸。
基本上視訊會議都是透過 WebRTC 來實現，幾乎所有即時互動功能都會用到 WebRTC，而常見的的遊戲直播，基本上延遲會比較高，主要使用的技術應該是串流 (Streaming) 來實現，這邊要注意一下。

## 運作原理

我們可以透過模擬兩位董事長要進行開會來學習 WebRTC 的運作原理：

1. 告知秘書準備要開會: 創建 PeerConnection 實例，並設定 ICE 連線通道的名單
2. 董事長請秘書把名片交給對方： 使用 SDP 交換彼此的資訊
3. 確認要在哪個會議室進行談話: 使用 ICE 協定來確認彼此的網路狀態
4. 雙方開始進行對話: 完成 RTC 連線，開始進行對話

### 告知秘書準備要開會

秘書就像是 PeerConnection 實例，我們需要先告知秘書要開會，秘書才會知道要準備哪些東西。

而 PeerConnection 實例的創建，通常會在準備建立連線時就創建，這樣才能確保後續交換資訊與連線順利進行。

那通常在創建的實例中我們需要提供什麼? 那就是可能會開會的地點 ICE Candidate，也就是雙方對話的通道

ICE 全名是 Interactive Connectivity Establishment，是一個用於在 WebRTC 中確認彼此的網路狀態的協定。

後續我們會在『確認要在哪個會議室進行談話』，介紹 ICE Candidate 選擇的順序。

### 董事長請秘書把名片交給對方

在對談中我們需要知道名字，對方講什麼語言，興趣是什麼，才能順利的進行對話，而這在 WebRTC 裡就是透過 SDP 來交換彼此的資訊。

SDP 全名為 Session Description Protocol，是一個用於描述媒體會話的協定，主要用於在 WebRTC 中交換彼此的資訊。

SDP 的格式如下
![SDP 格式](./sdp.png)

從以上可以知道這些資訊

```
📄 基本資訊
發起人 IP：10.47.16.5
會議名稱：SDP Seminar
主辦人信箱：j.doe@example.com
連線地址：224.2.17.12/127

🔊 音訊設定
傳輸協定與 Port：RTP/AVP over port 49170

🎥 視訊設定
傳輸協定與 Port：RTP/AVP over port 51372
```

所以 SDP 就像是我們要跟對方對話時，我們會先交換雙方的資訊，雙方才能根據彼此的編碼方式、媒體格式建立連線。

### 確認要在哪個會議室進行談話

就算我們交換了名片，但如果我們是在大馬路上交談，那很有可能會被汽車聲蓋過，所以需要找到一個適合的場所，例如咖啡廳就很棒。
而這在 WebRTC 裡就是透過 ICE 來確認彼此的網路狀態，並找到適合的通道，而這些通道在一開始就已經先設定好 (告知秘書準備要開會)
後續只是透過 ICE 來確認彼此的網路狀態，並找到適合的通道。

ICE 會將可以連線的通道稱為 Candidate，而 Candidate 可以分為以下幾種

- Host Candidate: 直接使用 IP 和 Port 進行連線，通常在區域網路(內網)中使用
- Server Reflexive Candidate：使用 STUN 伺服器協助取得 NAT 外可見的 IP 和 Port
- Relay Candidate：使用 TURN 伺服器中繼傳輸音訊與影像，是最穩但最慢的方式

通道的使用順序如下

1. 先使用 Host Candidate 進行連線
2. 如果 Host Candidate 連線失敗，則使用 Server Reflexive Candidate 進行連線
3. 如果 Server Reflexive Candidate 連線失敗，則使用 Relay Candidate 進行連線，通常 STUN 會失敗的原因是因為防火牆的關係，所以需要使用到 TURN 協定來進行連線。

視網路環境而定，但許多企業或教育網路環境常被防火牆阻擋，故最終會 fallback 到 TURN。

## 開始進行對話

到這邊基本上就是 ICE 連線完成後，雙方就可以看到彼此並進行對話

## 運作流程圖

實務上，SDP 和 ICE candidate 過程中可以分開傳遞（稱為 Trickle ICE），也可以合併傳遞（非 Trickle ICE）。

文字版

```
A 創建 RTCPeerConnection
B 創建 RTCPeerConnection
A 加入影音裝置 (getUserMedia & addTrack)
B 加入影音裝置 (getUserMedia & addTrack)
A 呼叫 createOffer → setLocalDescription
B 呼叫 setRemoteDescription(offer)
B 呼叫 createAnswer → setLocalDescription
A 呼叫 setRemoteDescription(answer)
A 呼叫 addIceCandidate(ice candidate)
B 呼叫 addIceCandidate(ice candidate)
ice 選擇對應通道並連線成功，雙方就可以開始進行對話
```

圖片版
![WebRTC 運作流程](./webrtc-flow.png)

## 小結

這邊純用前端做的話，只需要用到 PeerConnection 來進行溝通，但若是把溝通在後端進行實作的話會再多一個 Signaling Server (Websocket)，透過 Socket 將雙方的 SDP，ICE 連線狀況做傳遞。
所以 RTC 連線跟 Socket 連線是解耦的，只是因為現在為了方便控管，會讓他們的訊息傳遞都透過 Socket 來進行傳遞。
現在常見的 RTC SDK 也都是連線到提供方的 WebSocket 來進行傳遞，實際的 RTC 連線我們就碰觸不到了。

過去寫過的 RTC 連線實作可以參考 [WebRTC Demo](https://github.com/marshal604/webrtc-practice)
