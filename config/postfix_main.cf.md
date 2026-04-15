# Postfix 設定パラメータ一覧

`/etc/postfix/main.cf` に設定したパラメータの詳細解説です。

---

## 設定コマンド（実行手順）

```bash
postconf -e "myhostname = mail.local"
postconf -e "mydomain = local"
postconf -e "myorigin = \$mydomain"
postconf -e "inet_interfaces = all"
postconf -e "mydestination = \$myhostname, localhost.\$mydomain, localhost, \$mydomain"
postconf -e "home_mailbox = Maildir/"

systemctl restart postfix
```

---

## パラメータ解説

| パラメータ | 設定値 | 説明 |
|---|---|---|
| `myhostname` | `mail.local` | このメールサーバーのホスト名。メールヘッダーの `Received:` に使われる |
| `mydomain` | `local` | メールドメイン名。`user@local` の `local` 部分 |
| `myorigin` | `$mydomain` | 送信メールの差出人ドメイン。`$mydomain` を参照し `local` になる |
| `inet_interfaces` | `all` | メールを受け付けるネットワークインターフェース。`all` で全て受信 |
| `mydestination` | `$myhostname, localhost.$mydomain, localhost, $mydomain` | このサーバーが最終配送先として受け取るドメイン一覧 |
| `home_mailbox` | `Maildir/` | メールの保存形式。`Maildir/` でMaildir形式（Dovecotと連携必須） |

---

## 設定確認コマンド

```bash
postconf myhostname mydomain myorigin inet_interfaces mydestination home_mailbox
```

### 確認結果（実際の出力）

```
myhostname = mail.local
mydomain = local
myorigin = $mydomain
inet_interfaces = all
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
home_mailbox = Maildir/
```

---

## Maildir 形式について

`home_mailbox = Maildir/` を設定することで、メールが以下の構造で保存されます。

```
/home/testuser2/
└── Maildir/
    ├── new/   ← 未読メールが届く場所
    ├── cur/   ← 既読メール
    └── tmp/   ← 一時ファイル
```

Dovecot は Maildir 形式を読み取り、IMAP/POP3 でクライアントに提供します。

---

## デフォルト設定との違い

| パラメータ | デフォルト値 | 変更後 | 変更理由 |
|---|---|---|---|
| `myhostname` | `localhost.localdomain` | `mail.local` | 本番らしいホスト名に変更 |
| `mydomain` | `localdomain` | `local` | ローカルドメインに統一 |
| `myorigin` | `$myhostname` | `$mydomain` | 差出人ドメインをシンプルに |
| `inet_interfaces` | `localhost` | `all` | 外部からの受信を許可 |
| `home_mailbox` | （未設定） | `Maildir/` | Dovecot連携のために必須 |

---

## 関連ファイル

| ファイル | 用途 |
|---|---|
| `/etc/postfix/main.cf` | Postfixメイン設定ファイル |
| `/etc/postfix/master.cf` | Postfixサービス定義ファイル |
| `/var/spool/postfix/` | メールキューディレクトリ |
| `/var/log/maillog` | メールログ（またはjournalctl -u postfix） |

---

作成日：2026年4月15日  
作成者：長野 祐己
