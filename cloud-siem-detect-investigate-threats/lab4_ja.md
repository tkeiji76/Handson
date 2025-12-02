---
slug: capture-the-flag
type: challenge
title: Cloud SIEM Capture the Flag
teaser: '' 
notes:
- type: text
  contents: |
    サイバーセキュリティでは、Capture the Flag（CTF）ゲームはスキルを磨く素晴らしい方法です。目標は、侵害されたシステム内で「フラグ」または情報の一部を取得することです。

    Cloud SIEM CTFでは、Signals ExplorerとCloud SIEM Investigatorを使用して、ラボAWSアカウントの攻撃に関連する情報を見つけることができます。

    > **注意**: このラボは10分間操作がないとタイムアウトします。
tabs: 
- title: Terminal
  type: terminal
  hostname: lab-host
- title: IDE
  type: service
  hostname: lab-host
  path: /
  port: 8080
- title: AWS Console
  type: service
  hostname: cloud-client
  path: /
  port: 80
- title: Capture the Flag
  type: service
  hostname: lab-host
  path: /
  port: 7777
- title: Help
  type: website
  url: https://datadoghq.dev/training-lab-support?sandboxId=${_SANDBOX_ID}
difficulty: basic
---

Cloud SIEM Capture the Flag（オプション）
=== 
> [!NOTE]
> **Next**ボタンを使用してCapture the Flagアクティビティをスキップし、ラボを終了します。

Signals ExplorerとCloud SIEM Investigatorを使用して、ラボAWSアカウントの攻撃に関連する情報を見つけます。一部のフラグはIPアドレスやAWSリソース名などの値ですが、他のフラグは`[Flag-<flag_value>]`形式の文字列です。

**フラグを見つけるためにDatadogから離れる必要はありません。**

CTFをプレイするには、ラボナビゲーションの**[Capture the Flag tab](tab-3)**をクリックします。各フラグについて、ヒントを表示したり、答えを表示したりできます。すべてを見つけようとする際に、無制限に回答を送信できます。

フラグの見つけ方の例を見たい場合は、以下の折りたたまれたセクションを展開してください。

楽しんでください！ラボを終了する準備ができたら、**Next**ボタンを使用してください。

<Solutions>
<details>
<summary><strong>Flag 1 Solution</strong></summary>
<p><span><br><strong>malicious-iam-user</strong><br><br>Cloud SIEM Investigatorを使用して、作成されたIAMユーザーの名前を見つけます。<img src="../assets/04-capture-the-flag/question_1_malicious_iam_user.png" alt="Use Cloud SIEM Investigator to find the user that was created.">  </span></p>
</details>

<details>
<summary><strong>Flag 2 Solution</strong></summary>
<p><span><br><strong>TA0004-Privilege-Escalation</strong><br><br>この値を見つけるには複数の方法があります。<br> 1つの戦略は、Signals Explorerで文字列"AdministratorAccess"を検索して、関連するシグナルを見つけることです。<img src="../assets/04-capture-the-flag/question_2_find_admin_access_signal.png" alt="Find the relevant signal by searching for the string AdministratorAccess in Signals Explorer">シグナルの詳細を開いて、MITREタクティクスを見つけます。<img src="../assets/04-capture-the-flag/question_2_find_tactic.png" alt="Find the tactic in the signal."> </span></p>
</details>

<details>
<summary><strong>Flag 3 Solution</strong></summary>
<p><span><br><strong>GetCallerIdentity</strong><br><br>Cloud SIEM Investigatorを使用して、攻撃中に発生したstsイベントを見つけます。イベントタイプは1つだけです。<img src="../assets/04-capture-the-flag/question_3_sts_events.png" alt="Investigate STS events using Cloud SIEM Investigator."> </span></p>
</details>

<details>
<summary><strong>Flag 4 Solution</strong></summary>
<p><span><br><strong>us-east-1</strong><br><br>この値を見つけるには多くの方法があります。1つの方法は、CloudTrailイベントの関連ログを表示することです。<img src="../assets/04-capture-the-flag/question_4_related_logs_cloudtrail.png" alt="View related logs for the CloudTrail events."> DeleteTrailイベントログを選択します。<img src="../assets/04-capture-the-flag/question_4_view_logs.png" alt="Find the log for the delete trail event.">ログ内でリージョンを見つけます。<img src="../assets/04-capture-the-flag/question_4_region_in_log.png" alt="Find the log for the delete trail event."></span></p>
</details>

<details>
<summary><strong>Flag 5 Solution</strong></summary>
<p><span><br><strong>[Flag-CloudSIEM]</strong><br><br>Signals Explorerで高重要度の**New AWS account seen assuming a role into AWS account**シグナルを見つけます。<img src="../assets/04-capture-the-flag/question_5_find_high_severity_signal.png" alt="Find the high severity signal in Signals Explorer."> Playbookを読んでフラグを見つけます。<img src="../assets/04-capture-the-flag/question_5_flag_in_the_change_log.png" alt="Read the Rule Details to find the flag under the Changelog heading."> </span></p>
</details>





