## NHN Cloud > NHN Cloudリージョンガイド
リージョンは、独立していて地理的に隔離されたサーバーの物理的な位置を意味します。
一般的にリージョンは、アベイラビリティゾーンと呼ぶ独立した電源およびネットワークを持つデータセンターで構成され、使用したい地域とサービスに応じてリージョンを選択できます。 <br>
インターネットで、いつどこでも自由にリージョンを選択してNHN Cloudサービスを利用できます。

## NHN Cloudリージョン

NHN Cloudは、安定的なグローバルサービスを提供するために、4個のリージョンを運用しています。
高可用性をサポートするには、複数のアベイラビリティゾーンまたは複数のリージョンにアプリケーションを配布する必要があります。
NHN Cloudユーザーは、サービス地域と目的に応じて使用するリージョンを選択できます。一般的に主なサービス対象になる地域のリージョンを利用すると、短いレスポンス時間が期待できます。

## NHN Cloudリージョン位置
NHN Cloudは、グローバルなサービスを提供するために、多くの地域にリージョンを拡大しています。
![region_guide%2001.png](https://static.toastoven.net/toast/region_guide/Region_guide_2021.png)

## NHN Cloudリージョンサービス

**リージョンサービス**
リージョンサービスは、サービスを提供するリージョンのインフラ環境と国/地域/法律/商品でサービスする内容の制限により、特定地域にのみ提供されるサービスです。
物理的なサービスの提供位置が異なる必要がある場合や、データの二重化構成を行うためにも利用します。
リージョンサービスは、特定リージョンでのみ利用でき、リージョンごとに課金ポリシーが異なる場合があります。

**グローバルサービス**
グローバルサービスは、すべてのリージョンで使用できるサービスです。
すべてのユーザーに同じ機能とポリシー、安定性、ユーザビリティを提供し、すべてのリージョンを選択できます。

**グローバル/リージョン別提供サービス**

| 分類 | サービス名 | グローバル/リージョンサービス | 韓国(パンギョ)リージョン | 韓国(坪村)リージョン | 日本(東京)リージョン | 米国(カリフォルニア)リージョン |
| --- | ---- | :--------: | :-------: | :-------: | :-------: | :----------: |
| Compute | Instance | リージョン | O | O | O | O |
|  | GPU Instance | リージョン | O |  |  |  |
|  | Instance Template | リージョン | O | O | O | O |
|  | Image | リージョン | O | O | O | O |
|  | Auto Scale | リージョン | O | O | O | O |
|  | System Monitoring | リージョン | O | O | O | O |
| Container | Kubernetes | リージョン | O | O |  |  |
|  | Container Registry | リージョン | O | O |  |  |
| Network | VPC | リージョン | O | O | O | O |
|  | NAT Instance | リージョン |  | O  |  |  |
|  | Floating IP | リージョン | O | O | O | O |
|  | Security Groups | リージョン | O | O | O | O |
|  | Network ACL | リージョン | O | O | O | O |
|  | Network Interface | リージョン | O | O | O | O |
|  | 一般Load Balancer | リージョン | O | O | O | O |
|  | 専用Load Balancer | リージョン | O | O | O | O |
|  | 物理Load Balancer | リージョン | O | O |  |  |
|  | DNS Plus | グローバル |  |  |  |  |
| Storage | Block Storage | リージョン | O | O | O | O |
|  | NAS (offline) | リージョン | O | O  |  | O |
|  | Object Storage | リージョン | O | O | O | O |
|  | Backup | リージョン | O | O  | O |  |
|  | Data transporter | リージョン | O | O  |  |  |
| Database | RDS for MySQL | リージョン | O | O | O |  |
|  | RDS for MS-SQL | リージョン | O |  |  |  |
|  | EasyCache | リージョン | O | O | O |  |
|  | MS-SQL Instance | リージョン | O | O | O | O |
|  | MySQL Instance | リージョン | O | O | O | O |
|  | PostgreSQL Instance | リージョン | O | O | O  |O  |
| Game | Gamebase | グローバル |  |  |  |  |
|  | GameAnvil | グローバル |  |  |  |  |
|  | Leaderboard | グローバル |  |  |  |  |
|  | Launching | グローバル |  |  |  |  |
|  | Smart Downloader | グローバル |  |  |  |  |
| Security | NHN AppGuard | グローバル |  |  |  |  |
|  | App Security Check | リージョン | O |  |  |  |
|  | Server Security Check | リージョン | O | O |  |  |
|  | Security Monitoring | リージョン | O | O |  |  |
|  | Basic Security | リージョン | O |O |  |  |
|  | CAPTCHA | リージョン | O |  |  |  |
|  | OTP | リージョン | O |  |  |  |
|  | Web Firewall | リージョン | O | O  |  |  |
|  | Vaccine | リージョン | O | O |  |  |
|  | Secure Key Manager | グローバル |  |  |  |  |
|  | Security Compliance | グローバル |  |  |  |  |
|  | DDoS Guard | リージョン | O | O  |  |  |
|  | SIEM | リージョン | O |O |  |  |
| Content Delivery | CDN | グローバル |  |  |  |  |
|  | Image | リージョン | O |  |  |  |
| Notification | Push | グローバル |  |  |  |  |
|  | SMS | リージョン | O |  |  |  |
|  | Email | グローバル |  |  |  |  |
|  | KakaoTalk Bizmessage | リージョン | O |  |  |  |
| AI Service | Face Recognition | リージョン | O | O |  |  |
|  | AI Fashion | リージョン | O | O | O |  |  |
| Application Service | Maps | リージョン | O |  |  |  |
|  | ROLE | グローバル |  |  |  |  |
|  | API Gateway | リージョン | O |  |  |  |
|  | RTCS | グローバル |  |  |  |  |
|  | Short URL | グローバル |  |  |  |  |
|  | Cheating Detection | リージョン | O |  |  |  |
| Mobile Service | IAP | グローバル |  |  |  |  |
|  | Mobile Device Info | グローバル |  |  |  |  |
| Search | Cloud Search | リージョン | O |  |  |  |
|  | Autocomplete | リージョン | O |  |  |  |
|  | Corporation Search | リージョン | O |  |  |  |
|  | Address Search | リージョン | O |  |  |  |
| Analytics | Log & Crash Search | グローバル |  |  |  |  |
| Dev Tools | Pipeline | リージョン | O | O |  |  |
|  | Deploy | グローバル |  |  |  |  |
| Management | Managed | リージョン | O | O |  |  |
|  | Service Monitoring | グローバル |  |  |  |  |
|  | Certificate Manager | グローバル |  |  |  |  |
| Bill | eTax | リージョン | O |  |  |  |
| Dooray! | Project | グローバル |  |  |  |  |
|  | Messenger | グローバル |  |  |  |  |
|  | Mail | グローバル |  |  |  |  |
|  | Calendar | グローバル |  |  |  |  |
|  | Drive | グローバル |  |  |  |  |
|  | Contacts | グローバル |  |  |  |  |
|  | ウィキ | グローバル |  |  |  |  |
| Workplace \| ERP | Human resources | リージョン | O |  |  |  |
|  | Finance | リージョン | O |  |  |  |
| Workplace\| Groupware | Communication Board | リージョン | O |  |  |  |
|  | Workflow | リージョン | O |  |  |  |
| Contact Center | Omni Contact | リージョン | O |  |  |  |
|  | Mobile Contact | リージョン | O |  |  |  |
|  | Online Contact | グローバル |  |  |  |  |
| IDC | NCC | リージョン | O |  |  |  |
| CloudTail | CloudTail | グローバル |  |  |  |  |
