ラボ環境を探索する
===
ようこそ！このラボでは、Datadog Cloud SIEMを使用してAWS環境でリアルタイムの脅威を検知します。

あなたは新しいSAAS eラーニングプラットフォームのローンチに向けて懸命に取り組んでいるチームの一員だと想像してください。上司から、新しいアプリケーションとそれが動作するクラウド環境のセキュリティ監視を主導するよう依頼されました。

最近、Datadog Cloud SIEMをセットアップし、AWS CloudTrailログをDatadogに取り込んでセキュリティ分析を開始しました。また、CloudTrail用のCloud SIEM Content Packも設定しました。

Cloud SIEMが導入されたことに興奮しており、アプリが本番稼働する前に使用方法を練習したいと考えています。このラボ全体を通して、攻撃エミュレーションツールを使用して環境に対して攻撃を実行し、Cloud SIEMを使用して攻撃を検知・調査します。

ラボには2つのサンドボックスAWSアカウントが含まれています：

- **lab-aws-account**: このAWSアカウントは、あなたが保護する責任のあるアプリケーションをホストしています。ラボ全体を通して主にこのアカウントで作業します。
- **attacker-account**: このAWSアカウントは敵対者のAWSアカウントを表します。攻撃をシミュレートするために使用します。

ラボAWSアカウントにアクセスする
===

このラボ中、AWSコンソールで直接作業する必要はありませんが、次のいずれかの方法を使用してラボAWSアカウントにログインすることができます：

1. 次のコマンドを実行して、**[lab terminal](tab-0)** にサインインアドレスを表示します。サインインアドレスをクリックするか、ブラウザにコピー＆ペーストして、AWSコンソールを開いて認証します。

    ```run
    awsconsolelogin lab-aws-account
    ```

    > [!NOTE]
    > サインインURLを使用する際にエラーメッセージが表示される場合は、AWSアカウントに既にログインしていないことを確認してから、もう一度URLを試してください。

2. ラボターミナルの上にある**[AWS Console tab](tab-2)**をクリックして、AWSアカウントの認証情報を表示します。AWSコンソールまたは[AWS CLI](https://aws.amazon.com/cli/)を使用してクラウドアカウントにアクセスできます。

> [!NOTE]
> クラウドアカウントは一時的なもので、このラボ中のみ存続します。

Out-of-the-box検知ルール
===
最初の攻撃手法を実行する前に、Cloud SIEMが攻撃手法や疑わしい活動をフラグ付けするために使用するout-of-the-box（OOTB）検知ルールを探索します。

1. **[lab terminal](tab-0)**に表示されているDatadog認証情報を使用して、このラボ用の[Datadog](https://app.datadoghq.com)アカウントにログインします。必要に応じて、`creds`コマンドを実行して認証情報を再度表示します。

    ```run,bash
    creds
    ```

2. Datadogのメインメニューで、**Security**にカーソルを合わせます。**Cloud SIEM**の下にある**[Detection Rules](https://app.datadoghq.com/security/rules?query=type%3A%28log_detection%20OR%20signal_correlation%29&deprecated=hide)**をクリックします。

3. いくつかのコアファセットを使用してルールをグループ化およびソートできます。**Group by**セレクタを使用して、ルールを`tactic`でグループ化します。

    ![Group the detection rules by MITRE ATT&CK tactic](../assets/01-detonate-an-attack/group_by_tactic.png)

    Cloud SIEM Detection Rulesは、[MITRE ATT&CK](https://attack.mitre.org/)のタクティクスとテクニックにマッピングされています。

4. **Initial Access**セクションを展開して、環境への最初の足がかりを得ようとする攻撃者の試みをフラグ付けするOOTB検知ルールを確認します。

    ![Detection rules mapped to Initial Access tactic](../assets/01-detonate-an-attack/initial_access_detection_rules.png)

5. **Group by**ドロップダウンを**Source**に変更します。

    ![Detection rules grouped by rule source](../assets/01-detonate-an-attack/detection_rules_group_by_source.png)

    ルールが関連するソースとContent Packsでグループ化されるようになりました。

6. **cloudtrail**ルールのリストを展開します。これらのルールはCloudTrail Content Packから提供されています。

    ![cloudtrail detection rules](../assets/01-detonate-an-attack/cloudtrail_rules_expanded.png)

7. **Sort by**選択を**Highest Severity**に変更します。cloudtrailソースの最高重要度ルールを観察します。

    ![cloudtrail detection rules sorted by severity](../assets/01-detonate-an-attack/cloudtrail_rules_sorted_by_severity.png)

8. **Group by**設定を**Tactic**に戻します。

    ![rules grouped by tactic](../assets/01-detonate-an-attack/detection_rules_grouped_by_tactic.png)

9. **Tactic**ファセットを使用して検知ルールをフィルタリングすることもできます。ファセットリストの**MITRE ATT&CK**の下で、**TA0003-persistence**を選択します。

    ![rules filtered by persistence tactic](../assets/01-detonate-an-attack/detection_rules_filter_by_tactic.png)

10. **Source**ファセットに移動し、**cloudtrail**を選択してクエリフィルタをさらに絞り込みます。

    ![rules filtered by cloudtrail source](../assets/01-detonate-an-attack/detection_rules_filter_by_source.png)

11. [完全なクエリフィルタ](https://app.datadoghq.com/security/rules?query=type%3A%28log_detection%20OR%20signal_correlation%29%20tactic%3ATA0003-persistence%20source%3Acloudtrail%20&deprecated=hide&groupBy=tactic&sort=highest_severity)は次のようになります：

    ```copy
    type:(log_detection OR signal_correlation) tactic:TA0003-persistence source:cloudtrail 
    ``` 

    これらの検知ルールは、攻撃者がAWS環境への永続的なアクセスを維持しようとする試みに関連するCloudTrailアクティビティをフラグ付けします。

    ![query filter filtering cloudtrail rules for the persistence tactic](../assets/01-detonate-an-attack/detection_rules_complete_query_filter.png)

12. 特定のAWSサービスに関連するCloudTrail検知ルールを探索するには、サービス名をクエリフィルタ用語として使用できます。

    [クエリフィルタ](https://app.datadoghq.com/security/rules?query=type%3A%28log_detection%20OR%20signal_correlation%29%20tactic%3ATA0003-persistence%20source%3Acloudtrail%20%22lambda%22%20&deprecated=hide&groupBy=tactic&sort=highest_severity)に`"lambda"`を追加します。
    
    ```copy
    "lambda"
    ``` 
    
    攻撃者がAWS Lambdaサービスを悪用して永続性を獲得しようとしていることを示す可能性のあるOOTB検知ルールが表示されます。

    ![query filter filtering cloudtrail rules for the persistence tactic with the lambda query term](../assets/01-detonate-an-attack/add_lambda_to_query.png)

13. **AWS Lambda function modified by IAM user**ルールをクリックします。

14. **Edit Rule**をクリックして、ルールの詳細を確認します。

    ![Edit rule button for OOTB detection rule](../assets/01-detonate-an-attack/edit_rule.png)

15. **Define search queries**の下に、このルールを駆動するログクエリが表示されます。

    ![Query for the AWS Lambda function modified by IAM user rule](../assets/01-detonate-an-attack/aws_lambda_rule_query.png)

    `lambda_function_modified`クエリは、一致するログを見つけるためにいくつかのファセットを使用します。

    - `source:cloudtrail` ログは`cloudtrail`ログソースからのものでなければなりません
    - `@eventSource:lambda.amazonaws.com` CloudTrailログにはAWS Lambdaサービスからのイベントがなければなりません
    - `@evt.name:UpdateFunctionCode*` CloudTrailログには`UpdateFunctionCode*`に一致するイベント名が含まれている必要があります、`*`はワイルドカード文字です
    - `@userIdentity.type:IAMUser` イベントに関連するアクションを実行したIAMエンティティのタイプを示します。このクエリに一致するには、アクションはIAMユーザーによって実行されたものである必要があります（IAMロールや他のタイプのIAMエンティティではなく）
    - `-@userIdentity.invokedBy:(cloudformation.amazonaws.com OR "AWS Internal")` `-`は除外演算子です。イベントがクエリに一致するためには、`-`の後に続くすべてがイベントに含まれていてはいけません

16. **Set Conditions**の下に、このルールがどのようにセキュリティシグナルを生成するかが表示されます。

    ![Conditions for the AWS Lambda function modified by IAM user rule](../assets/01-detonate-an-attack/aws_lambda_rule_conditions.png)

    `lambda_function_modified > 0`条件が満たされると、このルールは`info`セキュリティシグナルでセキュリティシグナルを生成します。

    ラボシナリオでは、このルールが現在のニーズに適しているかどうかわかりません。複数のIAMユーザーが、大規模な製品ローンチに向けて頻繁にLambda関数を更新しています。このルールのセキュリティシグナルは、チームにとって有用ではありません。

    とはいえ、infrastructure as code（IaC）とIAMロールのみでLambda関数を管理したい将来には、確かに有用なルールになる可能性があります。
    
    このルールは、クローンして修正するのにも役立ちます。たとえば、予期しないIAMユーザーによって更新が行われた場合にシグナルを出すように、このルールを修正できます。

    今のところ、このOOTBルールがシグナルを生成するのを一時的に停止したいと考えています。

17. **Create a Suppression**見出しを見つけます。

    1つのオプションは、ルールがシグナルを生成すべきでない特定の条件を定義することで、生成されるシグナルを制限するサプレッションルールを作成することです。たとえば、Lambda関数の変更が許可されている既知のIAMユーザーによってこの検知ルールがトリガーされた場合に、シグナルを制限するサプレッションルールを作成できます。

    ![Suppression rules in rule configuration](../assets/01-detonate-an-attack/aws_lambda_rule_suppression_rules.png)

18. サプレッションクエリに基づいてシグナルをサプレスする代わりに、このルールがシグナルをまったく生成しないようにすることに決めました。次の手順を使用して検知ルールを無効にします：

    1. **Detection Rules**をクリックして、[Detection Rules Explorer](https://app.datadoghq.com/security/configuration/siem/rules?query=type%3A%28log_detection%20OR%20signal_correlation%29)に戻ります。

    2. 同じクエリを使用してルールをフィルタリングします。

        ```
        type:(log_detection OR signal_correlation) tactic:TA0003-persistence source:cloudtrail "lambda" 
        ```

    3. **AWS Lambda function modified by IAM user**ルールの横にあるトグルを使用して、ルールを無効にします。いつでもルールを再度有効にできます。

        ![Disable the unnecessary detection rule](../assets/01-detonate-an-attack/disable_lambda_modified_rule.png)

よくできました！Cloud SIEM検知ルールを正常に探索し、不要なルールを無効にしました。

検知ルールに慣れたので、設定をテストする準備が整いました！

Stratus Red Teamで攻撃を実行する
===

[Stratus Red Team](https://opensource.datadoghq.com/projects/stratus-red-team/)は、一般的なクラウド攻撃手法を再現するオープンソースの攻撃エミュレーターです。Stratus Red Teamはラボ環境に既にインストールされています。

1. **[lab terminal](tab-0)**で次のコマンドを実行して、Stratus Red Teamが再現できる攻撃手法のリストを表示します：

    ```run
    stratus list
    ```

    > [!NOTE]
    > 次のコマンドを使用して、AWS攻撃手法を具体的にリストすることができます：
    > ```run,bash
    > stratus list --platform aws
    > ```

    出力例（一部）：

    ```nocopy
    root@lab-host:~# stratus list

    View the list of all available attack techniques at: https://stratus-red-team.cloud/attack-techniques/list/

    +-------------------------------------------------------------+------------------------------------------------------------------+------------+----------------------+
    | TECHNIQUE ID                                                | TECHNIQUE NAME                                                   | PLATFORM   | MITRE ATT&CK TACTIC  |
    +-------------------------------------------------------------+------------------------------------------------------------------+------------+----------------------+
    | aws.credential-access.ec2-get-password-data                 | Retrieve EC2 Password Data                                       | AWS        | Credential Access    |
    | aws.credential-access.ec2-steal-instance-credentials        | Steal EC2 Instance Credentials                                   | AWS        | Credential Access    |
    | aws.credential-access.secretsmanager-batch-retrieve-secrets | Retrieve a High Number of Secrets Manager secrets (Batch)        | AWS        | Credential Access    |
    | aws.credential-access.secretsmanager-retrieve-secrets       | Retrieve a High Number of Secrets Manager secrets                | AWS        | Credential Access    |
    | aws.credential-access.ssm-retrieve-securestring-parameters  | Retrieve And Decrypt SSM Parameters                              | AWS        | Credential Access    |
    | aws.defense-evasion.cloudtrail-delete                       | Delete CloudTrail Trail      
    ```

攻撃を実行する
===

> [!IMPORTANT]
> 攻撃コマンドを実行する前に、CloudTrailログがトライアルのDatadogアカウントに到着していることを確認してください。次のリンクを使用して、[service:cloudtrail query](https://app.datadoghq.com/logs?query=service%3Acloudtrail&cols=host%2Cservice&fromUser=true&index=%2A&messageDisplay=inline&refresh_mode=sliding&storage=hot&stream_sort=desc&viz=stream&from_ts=1709680605784&to_ts=1709681505784&live=true)でログをフィルタリングできます。![CloudTrail logs in Datadog log explorer](../assets/01-detonate-an-attack/datadog_logs_cloudtrail_logs.png) 

このシミュレートされた攻撃では、外部のAWSアカウントの`attacker`ユーザーとして開始します。誤って設定されたIAMロールを利用して、ターゲットアカウントの`vulnerable-role` IAMロールを引き受けます。`vulnerable-role`を正常に引き受けた後、Stratus Red Teamを使用して一般的なクラウド攻撃のタクティクスをシミュレートします。

次の図は、このステージングされた攻撃の攻撃パスを示しています：

![Attack path for your staged attack](../assets/01-detonate-an-attack/attack_path.png)

次の手順に従って攻撃をステージングします：

1. 現在のラボ環境の次のAWSアカウントIDに注意してください：

    **lab-aws-account**: `[[ Instruqt-Var key="LAB_VAR_LAB_AWS_ACCOUNT_ID" hostname="lab-host" ]]`

    **attacker-account**: `[[ Instruqt-Var key="LAB_VAR_ATTACKER_ACCOUNT_ID" hostname="lab-host" ]]`

2. 次のコマンドを実行して、`attacker`として動作していることを確認し、ロールの**Arn**に注意してください。

    ```run
    aws sts get-caller-identity
    ```
    出力例：

    ![Expected output for sts command with account ID and attacker user Arn highlighted](../assets/01-detonate-an-attack/expected_output_sts_user.png)

    ラボ環境では、出力は次のようになります：

    ```nocopy
    {
    "UserId": "[[ Instruqt-Var key="LAB_VAR_ATTACK_USER_ID" hostname="lab-host" ]]",
    "Account": "[[ Instruqt-Var key="LAB_VAR_ATTACKER_ACCOUNT_ID" hostname="lab-host" ]]",
    "Arn": "arn:aws:iam::[[ Instruqt-Var key="LAB_VAR_ATTACKER_ACCOUNT_ID" hostname="lab-host" ]]:user/attacker"
    }
    ```

    `aws sts get-caller-identity`コマンドは、コマンドを実行するために使用されている認証情報のIAMユーザーまたはロールを返します。現在、外部の**attacker-account** AWSアカウントの`attacker`ユーザーとして動作しています。

3. `initial-access`攻撃タクティクスをエミュレートすることから始めます。次のコマンドを実行します：

    ```run
    eval $(aws sts assume-role --role-arn $VULNERABLE_IAM_ROLE_ARN --role-session-name "attacker-session" --query 'Credentials.[AccessKeyId,SecretAccessKey,SessionToken]' --output text | awk '{print "export AWS_ACCESS_KEY_ID="$1"\nexport AWS_SECRET_ACCESS_KEY="$2"\nexport AWS_SESSION_TOKEN="$3}')
    ```

    出力は期待されません。

    ![Image of the lab terminal with no output after running the command](../assets/01-detonate-an-attack/expected_output_assume_role.png)

    このコマンドで、外部の攻撃者がAWSラボアカウントの脆弱なIAMロールを引き受けた場合に何が起こるかをシミュレートしています。

    > [!NOTE]
    > このコマンドについて興味がある場合、次のことが起こっています：
    >  1. 攻撃者がロールを引き受けると、`attacker-session`という名前の一時セッションが作成されます。
    >  2. 一時的なAWSセッション認証情報が返されます。
    >  3. 一時セッション認証情報は、AWS CLIを設定するために使用される環境変数に保存されます。

4. `aws sts get-caller-identity`コマンドを再度実行し、ロールの**Arn**に注意してください。

    ```run
    aws sts get-caller-identity
    ```

    出力例：

    ![Expected output for the second sts get-caller-identity command with account ID and role arn highlighted](../assets/01-detonate-an-attack/expected_output_sts_role.png)

    ラボでは、コマンド出力は次のようになります：

    ```nocopy
    {
       "UserId": "<unique role ID>:attacker-session",
       "Account": "[[ Instruqt-Var key="LAB_VAR_LAB_AWS_ACCOUNT_ID" hostname="lab-host" ]]",
       "Arn": "arn:aws:sts::[[ Instruqt-Var key="LAB_VAR_LAB_AWS_ACCOUNT_ID" hostname="lab-host" ]]:assumed-role/vulnerable-role/attacker-session"
    }
    ```

    外部の`attacker` IAMユーザーは、`vulnerable-role`を使用してAWSラボアカウントでアクションを実行できるようになりました。

5. 攻撃者として最初のアクセスを獲得したので、次のコマンドを実行して`defense-evasion`タクティクスをエミュレートします：

    ```run
    stratus detonate aws.defense-evasion.cloudtrail-delete
    ```

    ![Expected terminal output for defense-evasion detonation. Stratus Red Team uses Terraform to spin up technique prerequisites. The output ends with confirmation a CloudTrail trail is being deleted](../assets/01-detonate-an-attack/expected_output_defense_evasion.png)

    出力例：

    ```nocopy
    root@lab-host:~# stratus detonate aws.defense-evasion.cloudtrail-delete
    2025/06/27 15:22:15 Checking your authentication against AWS
    2025/06/27 15:22:15 Creating /root/.stratus-red-team as it doesn't exist yet
    2025/06/27 15:22:15 Installing Terraform in /root/.stratus-red-team/terraform
    2025/06/27 15:22:16 Warming up aws.defense-evasion.cloudtrail-delete
    2025/06/27 15:22:16 Initializing Terraform to spin up technique prerequisites
    2025/06/27 15:22:21 Applying Terraform to spin up technique prerequisites
    2025/06/27 15:22:30 CloudTrail trail arn:aws:cloudtrail:us-east-1:637423184050:trail/stratus-red-team-cloudtraild-trail-cmhrelhfgz ready
    2025/06/27 15:22:30 Deleting CloudTrail trail stratus-red-team-cloudtraild-trail-cmhrelhfgz
    ```

    この手法は、攻撃者が検知を回避するためにCloudTrailトレイルを削除することをエミュレートしています。

    > [!NOTE]
    > `stratus detonate`コマンドの詳細については、[Stratus Red Team docs](https://stratus-red-team.cloud/user-guide/commands/detonate/)を参照してください。

6. 次のコマンドを実行して`persistence`タクティクスをエミュレートします：

    ```run
    stratus detonate aws.persistence.iam-create-admin-user
    ```
    ![Expected terminal output for persistence detonation. Stratus Red Team creates a malicious IAM user, attaches an administrative IAM policy, and creates a new access key](../assets/01-detonate-an-attack/expected_output_persistence.png)

    出力例：

    ```nocopy
    root@lab-host:~# stratus detonate aws.persistence.iam-create-admin-user
    2025/06/27 15:25:42 Checking your authentication against AWS
    2025/06/27 15:25:42 Creating a malicious IAM user
    2025/06/27 15:25:42 Attaching an administrative IAM policy to the malicious IAM user
    2025/06/27 15:25:42 Creating an access key for the IAM user
    2025/06/27 15:25:42 Created access key AKIAZI2LB5CZFBSKSRPI
    ```

    この手法は、攻撃者が新しいIAMユーザーを作成し、そのユーザーに権限ポリシーをアタッチすることをエミュレートしています。これで攻撃者はAWSアカウントへの永続的なアクセスを持つようになりました。

7. 次のコマンドを実行して`exfiltration`タクティクスをエミュレートします：

    ```run
    stratus detonate aws.exfiltration.ec2-security-group-open-port-22-ingress
    ```

    ![Expected terminal output for exfiltration detonation. Stratus Red Team uses Terraform to spin up technique prerequisites, creates a VPC, and opens port 22 on a security group](../assets/01-detonate-an-attack/expected_output_exfiltration.png)

    出力例：

    ```nocopy
    stratus detonate aws.exfiltration.ec2-security-group-open-port-22-ingress
    2025/06/27 15:28:27 Checking your authentication against AWS
    2025/06/27 15:28:27 Warming up aws.exfiltration.ec2-security-group-open-port-22-ingress
    2025/06/27 15:28:27 Initializing Terraform to spin up technique prerequisites
    2025/06/27 15:28:28 Applying Terraform to spin up technique prerequisites
    2025/06/27 15:28:39 Security group sg-0354d21d855914102 in VPC vpc-07eb1ef5dc3f4f46c ready
    2025/06/27 15:28:39 Opening port 22 from the Internet on sg-0354d21d855914102
    ```

    この手法は、攻撃者がセキュリティグループでポート22を開くことをエミュレートしています。

8. 最後に、次のコマンドを実行して`Java_Ghost`という名前のAWSセキュリティグループを作成します：

    ```run
    VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)
    aws ec2 create-security-group --group-name Java_Ghost --description "threat emulation security group" --vpc-id $VPC_ID
    ```

    ![Expected terminal output for java ghost security group command showing a GroupId beginning with sg](../assets/01-detonate-an-attack/expected_output_java_ghost.png)

    出力例：

    ```nocopy
    root@lab-host:~# VPC_ID=$(aws ec2 create-vpc --cidr-block 10.0.0.0/16 --query 'Vpc.VpcId' --output text)
    aws ec2 create-security-group --group-name Java_Ghost --description "threat emulation security group" --vpc-id $VPC_ID
    {
        "GroupId": "sg-0ce59fd9b8d273551"
    }
    ```

    Datadogのセキュリティ研究チームは、`Java_Ghost`セキュリティグループの作成がAWSにおける侵害されたアイデンティティを示すものであることを高い信頼度で評価しています。

この一連のコマンドを使用して、AWS環境への攻撃を正常にエミュレートしました。

具体的には、MITRE ATT&CKフレームワークの4つの異なるタクティクス（`initial-access`、`defense-evasion`、`persistence`、`exfiltration`）をシミュレートしました。また、Datadogのセキュリティ研究に基づいて、攻撃行動を示すAWSでのイベントもシミュレートしました。

攻撃パスのリマインダーは次のとおりです：

![Attack path showing each action you did in the above steps](../assets/01-detonate-an-attack/attack_path.png)

アクティビティのまとめ
===

このアクティビティでは、OOTB Cloud SIEM検知ルールをソートおよびレビューする方法を学びました。また、Stratus Red Teamを使用してラボAWSアカウントに対して攻撃をシミュレートしました。

攻撃をステージングしたので、次のラボアクティビティに進み、Cloud SIEMを使用して攻撃を検知し調査します。

**Next**ボタンを使用して次のラボアクティビティに進んでください。




