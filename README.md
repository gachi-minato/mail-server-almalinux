# 📧 メールサーバー構築ポートフォリオ
### Hyper-V × AlmaLinux 9.7 × Postfix × Dovecot

![AlmaLinux](https://img.shields.io/badge/AlmaLinux-9.7-0078C8?style=flat-square&logo=almalinux)
![Postfix](https://img.shields.io/badge/Postfix-3.5.25-red?style=flat-square)
![Dovecot](https://img.shields.io/badge/Dovecot-2.3.16-blue?style=flat-square)
![Hyper-V](https://img.shields.io/badge/Hyper--V-Windows11-0078D4?style=flat-square&logo=windows)

---

## 📋 概要

Windows 11 の Hyper-V 上に AlmaLinux 9.7 仮想マシンを構築し、  
Postfix（SMTP）と Dovecot（IMAP/POP3）を組み合わせたメールサーバーを一から構築したポートフォリオです。  
VSCode Remote SSH で接続しながら設定・動作確認まで実施しました。

---

## 🖥️ システム構成

```
【Windows 11 ホスト】
  └── Hyper-V
        └── AlmaLinux 9.7（IP: 192.168.3.12）
              ├── Postfix 3.5.25（SMTP: port 25）
              ├── Dovecot 2.3.16（IMAP: port 143 / POP3: port 110）
              └── firewalld 1.3.4
```

| 項目 | 内容 |
|---|---|
| ホストOS | Windows 11 Pro |
| 仮想化 | Hyper-V（第2世代VM） |
| ゲストOS | AlmaLinux 9.7 (Moss Jungle Cat) |
| VM スペック | CPU: 4コア / RAM: 4GB / Disk: 50GB |
| ネットワーク | External Switch（Wi-Fi共有） |
| 管理ツール | VSCode Remote SSH |

---

## 🛠️ 使用技術

| カテゴリ | 技術 |
|---|---|
| 仮想化 | Hyper-V（Windows標準） |
| OS | AlmaLinux 9.7（RHEL互換） |
| MTA（送信） | Postfix 3.5.25 |
| MDA（受信） | Dovecot 2.3.16 |
| ファイアウォール | firewalld 1.3.4 |
| パッケージ管理 | dnf |
| リモート管理 | VSCode Remote SSH |

---

## 🔧 構築手順

### Step 1 - Hyper-V 有効化
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

### Step 2 - AlmaLinux 9.7 インストール
- [AlmaLinux公式](https://almalinux.org/get-almalinux/) から DVD ISO を取得
- Hyper-V で第2世代VM を作成（セキュアブート無効）
- Minimal Install で インストール

### Step 3 - 初期設定・SSH接続
```bash
# tarインストール（VSCode Server展開に必要）
dnf install -y tar

# firewalld インストール・有効化
dnf install -y firewalld
systemctl enable --now firewalld
```

### Step 4 - firewalld ポート開放
```bash
firewall-cmd --permanent --add-service={smtp,smtps,imap,imaps,pop3,pop3s}
firewall-cmd --reload
```

### Step 5 - Postfix インストール・設定
```bash
dnf install -y postfix
systemctl enable --now postfix
```

```bash
# main.cf 設定
postconf -e "myhostname = mail.local"
postconf -e "mydomain = local"
postconf -e "myorigin = \$mydomain"
postconf -e "inet_interfaces = all"
postconf -e "mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain"
postconf -e "home_mailbox = Maildir/"

systemctl restart postfix
```

### Step 6 - Dovecot インストール
```bash
dnf install -y dovecot
systemctl enable --now dovecot
```

---

## ✅ 動作確認結果

### テストユーザー作成
```bash
useradd -m testuser1 && echo "testuser1:Password123" | chpasswd
useradd -m testuser2 && echo "testuser2:Password123" | chpasswd
```

### メール送信テスト（Postfix）
```bash
sendmail -v -f testuser1@local testuser2@local << EOF
From: testuser1 <testuser1@local>
To: testuser2 <testuser2@local>
Subject: Hello from testuser1

testuser1からのテストメールです。
EOF
```
**結果：** `/home/testuser2/Maildir/new/` にメールファイルの着信を確認 ✅

---

### IMAP テスト（Dovecot / port 143）
```
telnet localhost 143
a1 LOGIN testuser2 Password123   → a1 OK Logged in
a2 SELECT INBOX                  → * 1 EXISTS
a3 FETCH 1 BODY[]                → メール本文取得成功
a4 LOGOUT
```
**結果：** IMAP経由でメール取得成功 ✅

---

### POP3 テスト（Dovecot / port 110）
```
telnet localhost 110
USER testuser2     → +OK
PASS Password123   → +OK Logged in.
STAT               → +OK 1 349
RETR 1             → +OK 349 octets（メール本文取得）
QUIT
```
**結果：** POP3経由でメール取得成功 ✅

---

### ログ確認（journalctl）
```bash
journalctl -u postfix --since "2026-04-15 13:00:00" --no-pager
```
```
postfix/local: status=sent (delivered to maildir)  ✅
postfix/local: status=sent (delivered to maildir)  ✅
```
**結果：** 2通ともMaildir配送成功をログで確認 ✅

---

## 📂 リポジトリ構成

```
mail-server-almalinux/
├── README.md                        # 本ファイル
├── docs/
│   ├── mailserver_design_v1.2.docx  # 基本設計書
│   └── mailserver_portfolio_v3.pptx # ポートフォリオスライド
└── config/
    └── postfix_main.cf.md           # Postfix設定パラメータ一覧
```

---

## 💡 アピールポイント

- **Microsoft技術（Hyper-V）** を活用したオンプレミス仮想化環境の構築
- **RHEL互換OS（AlmaLinux）** の構築・運用経験（企業サーバーで主流）
- **SMTP / IMAP / POP3** すべてのプロトコルで動作確認済み
- **firewalld** による最小権限のポート管理
- **VSCode Remote SSH** による快適なリモート管理環境
- telnetによるプロトコルレベルでの動作検証まで実施

---

## 👤 作成者

**長野 祐己**｜ インフラエンジニア  
作成日：2026年4月15日
