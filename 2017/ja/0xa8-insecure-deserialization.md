# A8:2017 安全ではないデシリアライゼーション

| 脅威エージェント/攻撃ベクター | セキュリティ上の弱点           | 影響度               |
| -- | -- | -- |
| アクセス Lvl : 悪用難易度 1 | 流行度 2 : 攻撃検知のしやすさ 2 | 技術的 3 : ビジネス的 |
| 市販されているエクスプロイト手法は、元のエクスプロイトコードに変更や調整を加えずに攻撃が成功する事はまれなので、デシリアライゼーションの悪用は、少々困難です。 | この問題は、OWASPが行った[業界調査](https://owasp.blogspot.com/2017/08/owasp-top-10-2017-project-update.html)において上位10に含まれていたものですが、定量的なデータに基づいたものではありません。 ツールによっては、デシリアライゼーションに関する欠陥を発見する事ができますが、問題を検証する為に、しばしば人手による支援を必要とします。 デシリアライゼーションに関する欠陥が、広く広まっているというデータは、問題の特定と対応を支援するツールが開発された事から予想されたものでした。 | デシリアライゼーションの欠陥による影響は、憂慮すべきものです。 これらの欠陥は、最も深刻な攻撃の一つであるリモートコード実行攻撃を可能にします。 ビジネスへの影響は、アプリケーションとデータを保護する必要性に依存します。 |

## アプリケーションは脆弱か？

攻撃者により敵意のあるデシリアライズまたは改ざんされたオブジェクトの供給により、アプリケーションとAPIは、脆弱になる。

これによる攻撃は、主に2種類ある:

* オブジェクトとデータ構造に関連した攻撃：デシリアライズ中またはデシリアライズ後に、振る舞いを変更できるクラスがアプリケーションで使用可能な場合、攻撃者は、アプリケーションロジックの変更または、任意のリモートコード実行を行う事が出来る。
* 典型的なデータ改ざん攻撃：既存のデータ構造が内容を変えられて使われるようなアクセス制御関連の攻撃

シリアライゼーションが、以下のような用途にアプリケーションで使用される場合：

* リモートプロシージャコールとプロセス間通信 (RPC/IPC) 
* ワイヤプロトコル、Webサービス、メッセージブローカ
* キャッシュ/永続化
* データベース、キャッシュサーバ、ファイルシステム
* HTTPクッキー、HTMLフォームのパラメータ、API認証トークン

## 対策

安全なアーキテクチャを実現するには、信頼できないソースでシリアライズされたオブジェクトを受け入れない、もしくは、シリアライズ対象のデータをプリミティブなデータ型のみにする。

上記の対策を取れない場合、以下の内、一つ以上を検討する：

* 敵対的なオブジェクトの生成やデータの改ざんを防ぐために、シリアライズされたオブジェクトにデジタル署名などの整合性チェックを実装する。
* コードは、定義可能なクラスを想定している為、オブジェクトを生成する前に、デシリアライゼーションにおいて厳密な型制約を強制する。ただし、この手法を回避する方法は実証済みなので、この手法頼みにする事はお勧め出来ない。
* 悪意あるコードが埋め込まれていた場合に備え、可能であればデシリアライズに関するコードは分離して、低い権限の環境下で実行する。
* 型の不整合やデシリアライズ時に生じた例外など、デシリアライゼーションで発生した失敗や例外はログに記録する。
* デシリアライズするコンテナやサーバーからの、送受信に関するネットワーク接続は、制限もしくはモニタリングする。
* ユーザーが絶えずデシリアライズする事がないか、デシリアライゼーションをモニタリングし、警告する。


## 攻撃シナリオの例

**シナリオ #1**: Reactアプリケーションは、Spring Bootマイクロサービスを呼び出す。
優秀なプログラマであろうとする為、彼らのコードはイミュータブルであろうとする。
彼らが思いついた解決策は、呼び出しの前後でシリアライズしたユーザーの状態を渡す。
攻撃者は base64でエンコードされている事を示す"R00"と言うJavaオブジェクトのシグネチャに気づき、Java Serial Killerツールを使用してアプリケーションサーバー上でリモートコードを実行する。

**シナリオ #2**: あるPHPフォーラムでは、PHPオブジェクトのシリアライゼーションを使用して、ユーザーのユーザーID、ロール、パスワードハッシュやその他の状態を含むSuper Cookieを保存：

`a:4:{i:0;i:132;i:1;s:7:"Mallory";i:2;s:4:"user";i:3;s:32:"b6a8b3bea87fe0e05022f8f3c88bc960";}`

攻撃者は、シリアライズされたオブジェクトを変更して攻撃者自身に管理者権限を与える。

`a:4:{i:0;i:1;i:1;s:5:"Alice";i:2;s:5:"admin";i:3;s:32:"b6a8b3bea87fe0e05022f8f3c88bc960";}`

## 参照

### OWASP

* [OWASP Cheat Sheet: Deserialization](https://www.owasp.org/index.php/Deserialization_Cheat_Sheet)
* [OWASP Proactive Controls: Validate All Inputs](https://www.owasp.org/index.php/OWASP_Proactive_Controls#4:_Validate_All_Inputs)
* [OWASP Application Security Verification Standard: TBA](https://www.owasp.org/index.php/Category:OWASP_Application_Security_Verification_Standard_Project#tab=Home)
* [OWASP AppSecEU 2016: Surviving the Java Deserialization Apocalypse](https://speakerdeck.com/pwntester/surviving-the-java-deserialization-apocalypse)
* [OWASP AppSecUSA 2017: Friday the 13th JSON Attacks](https://speakerdeck.com/pwntester/friday-the-13th-json-attacks)

### OWASP外

* [CWE-502: Deserialization of Untrusted Data](https://cwe.mitre.org/data/definitions/502.html)
* [Java Unmarshaller Security](https://github.com/mbechler/marshalsec)
* [OWASP AppSec Cali 2015: Marshalling Pickles](http://frohoff.github.io/appseccali-marshalling-pickles/)
