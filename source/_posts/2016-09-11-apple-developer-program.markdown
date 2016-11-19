---
layout: post
title: "Apple Developer Program(チーム活用編)"
date: 2016-09-11 20:08
comments: true
categories: ios
---

###はじめに
本日はいつもと少し違う視点でブログを書きたいと思います。  
今回取り上げるのは『チームでのApple Developer Programの活用方法』です。  
iPhoneが世に出てから数年の月日が経っているものの、未だにApple Developer Programの使い方が成立していないところも多いです。  
そこで、筆者が考える手法について備忘録の意を含めて書きたいと思います。

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

<!-- more -->

###Apple Developer Programとは
まずはApple Developer Programを簡単に説明していきます。  
Apple Developer ProgramのMembershipを購入することで下記のことができるようになります。

* iPhone, iPad, Apple Watch, Mac用のアプリを世界中に配信できます  
* Appleが開発者向けに提供しているAppの高度な機能を利用できます  
* 一般公開されていない最新のβ版OSを利用できます  
* TestFlightを利用することで公開前のアプリをテスト利用することができます  
* App Analyticsを利用することでアプリのインストール数, 利用頻度, 収益などを確認できます  

また、Apple Developer Programには下記2つの登録種別があります。  

* 個人として登録する  
* 法人として登録する  

さらに法人として利用する場合は次の2つのMembershipが存在します。  

* Apple Developer Program: 基本的には個人のApple Developer Programと同じ  
* Apple Developer Enterprise Program: 従業員のみを対象として設計および配布する独自の App を作成する場合に利用  

###Apple Developer Programでの役割
次に、Apple Developer Programでの役割と権限について説明します。  
個人契約の場合は役割や権限が存在しないため、法人契約視点で話します。  

Apple Developer Programでの役割は次の3種類です。  

* Team Agent: 登録者(契約者)本人で、チームに唯一の存在です  
* Admin: 契約関連の作業以外の全てを扱うことができます  
* Member: 開発に必要な最低限の作業のみを許されています  

具体的に各役割で何ができるのか見ていきましょう。  

|　　|　Team Agent　|　Admin　|　Member　|
|:-|-:|:-:|
|法的な契約の締結|　◯ |　☓ |　☓
|Membershipの更新|　◯ |　☓ |　☓
|Developer ID 証明書の作成|　◯ |　☓ |　☓
|Memberの招待と役割の割り当て|　◯ |　◯ |　☓
|Provisioning Profileの作成|　◯ |　◯ |　☓
|証明書署名リクエストの承認|　◯ |　◯ |　☓
|UDID の追加と無効化|　◯ |　☓ |　☓
|App ID の登録、設定、削除|　◯ |　◯ |　☓
|iOS配布用証明書と配布用Provisioning Profileの作成|　◯ |　◯ |　☓
|APNS証明書とPass Type IDの作成|　◯ |　◯ |　☓
|Mac App / Mac Installer配布証明書の作成|　◯ |　◯ |　☓
|Technical Support Incident の購入と送信|　◯ |　◯ |　◯
|Developer Forums への投稿|　◯ |　◯ |　◯
|プレリリース版ソフトウェアのDL|　◯ |　◯ |　◯
|Provisioning ProfileのDL|　◯ |　◯ |　◯
|証明書署名リクエストの送信|　◯ |　◯ |　◯
|Safari 拡張機能証明書の作成|　◯ |　◯ |　◯
|Safari Extensions GalleryへのSafari拡張機能の提出|　◯ |　◯ |　◯

ここでポイントとなるのは(実は先にも述べていたのですが)、 **Admin** は **ほとんどの作業を許可されている** ということです。  

###MemberCenterでできること(iOS, tvOS, watchOS)
上記で説明した役割をどのように組織やチームに割り振るのかを考える前に、Apple Developer Programの管理サイトであるMemberCenterで何ができるのかを見ていきます。  
今回はiOS, tvOS, watchOS共通の操作のみを説明します。  

MemberCenterでできることは下記です。  

* Certificatesの作成/DL/削除  
* アプリ関連の固有IDの登録/DL/編集/削除  
  * App IDsなど一部自由には削除できないものあり  
* 端末の登録/編集/削除  
  * 削除は年1回のみ可能  
* Provisioning Profilesの登録/DL/編集/削除  

実際の画面のメニューは次のとおりです。  

![Apple Developer ProgramのMember Centerのメニュー](/images/apple_member_center_menu.png)  

####Certificatesについて
CertificatesはMemberCenterにサインインしたアカウントに紐づきます。  
Development系の証明書に関しては大きな影響はないものの、Production系の証明書に関してはリリースに影響が出るため慎重に扱う必要があります。  
しかも、このCertificatesはTeam AgentのみならずAdminも作成と削除が可能になっています。  
法人としてアプリをリリースする場合、Production用の **App Store and Ad Hoc** のCertificatesは統一します。(個人アカウント名義での証明書でリリースするわけにはいかないためです。)  
Developmentはあくまでも開発用に利用するため統一の必要はありません。  

####App IDsについて
アプリ関連の固有IDの中で **App IDs** というものがあります。  
これはアプリ固有のIDであり、Development用とProduction用を兼ねて扱います。  
Push用の証明書もDevelopment用とProduction用の2つを1つの **App IDs** に紐付けることができます。  

これも安易に削除できないため不要なものを作り過ぎないようにしましょう。  

####Devicesについて
iPhone, iPadなどの端末を登録します。  
最大登録数が100台と決まっているため、登録する端末は厳選する必要があります。  
端末の削除は1年に1回であり、例外は認められていません。  

Xcode7からここで登録していない端末でも開発用のアプリをインストールすることが可能になっていますが、利用可能な機能に制限があります。  
例えば、Push通知の動作確認をするためにはここで登録する必要があります。  

####Provisioning Profilesについて
これは簡単に言えば、アプリ単位でのビルド用証明書です。  
編集や削除は簡単にできますので、MemberCenterの中でも比較的、気兼ねせずに利用できる機能となっています。  

###組織での権限の割り振り方について
さて、ここまでくれば、既に何となく想像できているかもしれません。  
しかしながら、Appleが定義している役割はTeam Agent / Admin / Memberの3つのみであるため、権限の差異はかなり大雑把なものとなっています。  
そのため、ある程度、組織内でのルールを定義して扱わざるを得ません。  

筆者が提案したい組織内での権限の割り振り方は2つあります。  

####強固なパターン
まずは強固なパターンを説明します。  

* Team Agent  
  * 契約に関する作業を実施  
  * Production用のCertificatesの作成  
  * 上記以外で依頼されたCertificatesの作成
  * 依頼されたApp IDsの作成  
  * 依頼された端末の登録  
  * 依頼されたProvisioning Profilesの作成  
* Admin
  * この権現を持つアカウントは登録しない(この権限は利用しません)  
* Member
  * App IDsの作成をTeam Agentに依頼する
  * 端末の登録をTeam Agentに依頼する  
  * Provisioning Profilesの作成をTeam Agentに依頼する  

Team AgentとAdminの権限は契約以外同じであるため、いっそのことAdminを利用しないことで、オペレーションミスを回避することができます。  
証明書は安易に削除されてはよくないですし、登録数が限られているにも関わらず無造作に端末が登録されてしまうことも非常にリスクが高いと考えられます。  

####緩和なパターン
次に緩和なパターンを説明します。  

* Team Agent  
  * 契約に関する作業を実施  
  * Production用のCertificatesの作成  
  * (開発者かつ自身のプロジェクトを持つ場合)上記以外のCertificatesの作成  
  * (開発者かつ自身のプロジェクトを持つ場合)自身のプロジェクト用のApp IDsの作成  
  * (開発者かつ自身のプロジェクトを持つ場合)自身のプロジェクトに必要な端末の登録  
  * (開発者かつ自身のプロジェクトを持つ場合)自身のプロジェクト用のProvisioning Profilesの作成  
* Admin 
  * プロジェクトごとに代表者を1名選出してAdminとする  
  * Production用以外の自身のプロジェクト用のCertificatesの作成  
  * 自身のプロジェクト用のApp IDsの作成  
  * 自身のプロジェクトに必要な端末の登録  
  * 自身のプロジェクト用のProvisioning Profilesの作成  
* Member  
  * 自身のプロジェクトに必要なCertificates, App IDs, Provisioning ProfilesのDL  

Team Agentは契約作業もしますが、開発者が兼ねている場合は基本的にAdminと同じ扱いで考えます。  
Adminは『Production用以外の自身のプロジェクト用のCertificatesの作成』であれば許可し、あくまでも『Production用のCertificatesの作成』はTeam Agentが担います。  
流石に強力な権限を持つAdminを乱立させるのは危険であるため、プロジェクト内で代表者を選出するのが良いでしょう。  
同プロジェクト内の他の開発者はMemberで十分なはずです。  

###まとめ
如何でしたでしょうか？  
実際の様々な企業ではそれぞれの管理の仕方があるのかもしれませんが、筆者はどちらかというと強固なパターンで管理した方が良い派です。  
なぜなら、iOSアプリ開発者でも、MemberCenterの仕組みや証明書を理解している人は案外多くないからです。  
正しく理解した人が全てのアプリに影響を与えない形で管理することが結局は皆に安心安全な仕組みを提供することができるのです。  

と言ったところで本日はここまで。  

<script async src="//pagead2.googlesyndication.com/pagead/js/adsbygoogle.js"></script>
<ins class="adsbygoogle"style="display:inline-block;width:320px;height:100px"data-ad-client="ca-pub-2881241309408290"data-ad-slot="6603059167"></ins>
<script>
(adsbygoogle = window.adsbygoogle || []).push({});
</script>

