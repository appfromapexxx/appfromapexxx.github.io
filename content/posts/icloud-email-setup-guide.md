---
title: 實戰教學：使用 Cloudflare 網域與 iCloud+ 打造自訂 Email
date: 2025-12-10
draft: false
categories:
  - 架站實作
  - 技術寫作
tags:
  - apple-生態系
summary: 本教學將引導你如何利用 Cloudflare 的批發價網域服務，結合 Apple iCloud+ 的「自訂電子郵件網域」功能，建立專業的個人信箱。
showToc: true
tocOpen: true
---

本教學將引導你如何利用 **Cloudflare** 的批發價網域服務，結合 **Apple iCloud+** 的「自訂電子郵件網域」功能，建立專業的個人信箱（例如 `me@yourdomain.com`）。

此方案的優勢在於：Cloudflare 提供企業級 DNS 解析且無溢價續費，而 iCloud+ 則是許多 Apple 用戶已訂閱的服務，無須額外付費購買 Google Workspace 或 Microsoft 365 等企業信箱。

---

## 前置需求

1.  **Apple ID**：必須訂閱 **iCloud+** (50GB 方案以上即可)。
2.  **Cloudflare 帳號**：並已完成信用卡綁定（用於購買網域）。
3.  **雙重驗證**：建議兩個帳號都開啟 2FA 以確保安全。

---

## 第一階段：在 Cloudflare 購買網域

若你尚未持有網域，請先透過 Cloudflare Registrar 購買。

1.  登入 [Cloudflare Dashboard](https://dash.cloudflare.com/)。
2.  在左側選單點擊 **Domain Registration (網域註冊)** -> **Register Domain (註冊網域)**。
3.  搜尋你想要的網域（例如 `your-domain-name`）。
4.  選擇 `.com` 或其他頂級域名並完成結帳。
5.  **確認狀態**：購買完成後，確保該網域在 Cloudflare 中顯示為 "Active" (有效)，且 Nameservers (名稱伺服器) 預設即為 Cloudflare 代管。

---

## 第二階段：在 iCloud 後台發起設定

Apple 與 Cloudflare 有 API 串接合作，設定 DNS 紀錄的過程可完全自動化。

1.  使用瀏覽器登入 [iCloud.com](https://www.icloud.com/) 並進入 **帳號設定 (Account Settings)**。
2.  在「自訂電子郵件網域 (Custom Email Domain)」區塊，點擊 **管理 (Manage)**。
3.  選擇 **「僅限您使用 (Only You)」**（若要與家庭共享成員共用則選另一項）。
4.  輸入你剛購買的網域：`yourdomain.com`。

---

## 第三階段：授權 Cloudflare 自動設定 DNS

這是最關鍵的一步，系統會自動寫入 MX、SPF、DKIM 與 CNAME 紀錄。

1.  iCloud 會偵測到你的網域託管於 Cloudflare，並跳出提示視窗。
2.  點擊 **登入 (Log in)** 或 **Authorize (授權)**。
3.  瀏覽器會導向至 Cloudflare 登入頁面。
4.  登入後，畫面會顯示 Cloudflare 將授權 Apple 修改該網域的 DNS 區域檔。
5.  點擊 **Authorize (授權)**。
6.  等待幾秒鐘，iCloud 畫面會顯示「網域已驗證成功」。

> **注意**：若你原本有設定其他郵件服務（如原有 MX 紀錄），此動作通常會覆蓋或新增紀錄。由於是新網域，直接覆蓋即可。

---

## 第四階段：建立 Email 地址

網域驗證通過後，你需要建立實際的收發信地址。

1.  在 iCloud 的設定引導中，點擊 **「新增電子郵件地址」**。
2.  在 `@` 前面的欄位輸入：`me` (或是 `contact`, `hi` 等)。
3.  完整地址顯示為：`me@yourdomain.com`。
4.  點擊 **加入**。
5.  完成後，你可以選擇是否將此地址用於 FaceTime 或 iMessage（依個人需求）。

---

## 第五階段：設定預設寄件人 (Default Sender)

為了確保你撰寫新信件時，預設使用專業網域而非原本的 `@icloud.com`，需進行以下設定。

### 方法 A：透過 iCloud 網頁版 (全平台通用)

1.  回到 [iCloud Mail](https://www.icloud.com/mail/) 介面。
2.  點擊左上角的 **齒輪圖示 (Settings)** -> **偏好設定 (Preferences)**。
3.  切換到 **撰寫 (Composing)** 分頁。
4.  找到 **「設定預設寄件地址 (Set a default address for sending)」**。
5.  在下拉選單中選擇：`me@yourdomain.com`。

### 方法 B：透過 iPhone (iOS 17+)

1.  進入 **設定 (Settings)**。
2.  點擊上方你的 **Apple ID 名稱** -> **iCloud**。
3.  點擊 **iCloud 郵件 (iCloud Mail)**。
4.  點擊 **iCloud 郵件設定 (iCloud Mail Settings)**。
5.  在「iCloud 郵件地址」或「寄件人」選項中，確認 `me@yourdomain.com` 已被啟用。
6.  回到上一層，在 **「允許傳送來源 (Allow Sending From)」** 確保你的自訂網域已勾選。
7.  **(重要)** 進入 iOS 系統的 **設定** -> **郵件 (Mail)** -> 捲動至最下方的 **預設帳號 (Default Account)**，確認選擇 iCloud，並在 **總是使用預設帳號寄信** 的相關設定中留意寄件人顯示。通常 iOS 郵件 App 會記憶你上次使用的寄件人，或需手動在寫信畫面點擊寄件人欄位切換。

---

## 常見問題排查

* **DNS 傳播時間**：雖然 Cloudflare 更新極快，但全球 DNS 傳播有時仍需 15-30 分鐘。若無法立即收信，請稍作等待。
* **SPF 驗證**：若你日後透過第三方電子報系統（如 Mailchimp）使用此網域寄信，記得回到 Cloudflare DNS 設定中，修改 SPF 紀錄（TXT 紀錄），將第三方服務 include 進去，以免被歸類為垃圾信。