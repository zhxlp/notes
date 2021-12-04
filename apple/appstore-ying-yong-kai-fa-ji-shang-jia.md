# AppStore 应用开发及上架

## 创建证书签名请求

1. 启动位于 `/Applications/Utilities` 中的“钥匙串访问”。

   ![image-20200418130012838](../.gitbook/assets/image-20200418130012838.png)

2. 选取“钥匙串访问”>“证书助理”>“从证书颁发机构请求证书”。

   ![image-20200418130139000](../.gitbook/assets/image-20200418130139000.png)

3. 在“证书助理”对话框中，在“用户电子邮件地址”栏位中输入电子邮件地址。

4. 在“常用名称”栏位中，输入密钥的名称 (例如，Gita Kumar Dev Key)。

5. 将“CA 电子邮件地址”栏位留空。

6. 选取“存储到磁盘”，然后点按“继续”。

![image-20200418124740963](../.gitbook/assets/image-20200418124740963.png)

## 创建开发证书

1. 在“[Certificates, Identifiers & Profiles (英文)](https://developer.apple.com/account/resources)”(证书、标识符和描述文件) 中，从侧边栏中选择“Certificates”(证书)。

2. 在右上角，点按添加按钮 (+)。

   ![image-20200418132811297](../.gitbook/assets/image-20200418132811297.png)

3. 在“Software”(软件) 下面，选择“Apple Development”，然后点按“Continue”(继续)。

   ![image-20200418130631272](../.gitbook/assets/image-20200418130631272.png)

4. 点按“Choose File”(选取文件)。

   ![image-20200418130714522](../.gitbook/assets/image-20200418130714522.png)

5. 在出现的对话框中，选择证书请求文件 (创建证书签名请求中生成的文件)，然后点按“Choose”(打开)。

   ![image-20200418130926505](../.gitbook/assets/image-20200418130926505.png)

6. 点按“Continue”(继续)。

   ![image-20200418131107413](../.gitbook/assets/image-20200418131107413.png)

7. 点按“Download”(下载)。

   ![image-20200418131410295](../.gitbook/assets/image-20200418131410295.png)

8. 双击已下载的证书文件，在添加证书界面中，“钥匙串”选择“登录”，再点击“添加”

   ![image-20200418132022409](../.gitbook/assets/image-20200418132022409.png)

9. 证书可在“钥匙串访问”工具中查看

   ![image-20200418132632402](../.gitbook/assets/image-20200418132632402.png)

## 创建分发证书

1. 在“[Certificates, Identifiers & Profiles (英文)](https://developer.apple.com/account/resources)”(证书、标识符和描述文件) 中，从侧边栏中选择“Certificates”(证书)。

2. 在右上角，点按添加按钮 (+)。

   ![image-20200418132811297](../.gitbook/assets/image-20200418132811297.png)

3. 在“Software”(软件) 下面，选择“Apple Distribution”，然后点按“Continue”(继续)。

   ![image-20200418132944222](../.gitbook/assets/image-20200418132944222.png)

4. 点按“Choose File”(选取文件)。

   ![image-20200418133035778](../.gitbook/assets/image-20200418133035778.png)

5. 在出现的对话框中，选择证书请求文件 (创建证书签名请求中生成的文件)，然后点按“Choose”(打开)。

   ![image-20200418130926505](../.gitbook/assets/image-20200418130926505.png)

6. 点按“Continue”(继续)。

   ![image-20200418133210744](../.gitbook/assets/image-20200418133210744.png)

7. 点按“Download”(下载)。

   ![image-20200418133253117](../.gitbook/assets/image-20200418133253117.png)

8. 双击已下载的证书文件，在添加证书界面中，“钥匙串”选择“登录”，再点击“添加”

   ![image-20200418133413527](../.gitbook/assets/image-20200418133413527.png)

9. 证书可在“钥匙串访问”工具中查看

   ![image-20200418133608903](../.gitbook/assets/image-20200418133608903.png)

## 注册 App ID

1. 在“[Certificates, Identifiers & Profiles (英文)](https://developer.apple.com/account/resources)”(证书、标识符和描述文件) 中，从侧边栏中选择“Identifiers”(标识符)，然后点按左上角的添加按钮 (+)。

   ![image-20200418133828001](../.gitbook/assets/image-20200418133828001.png)

2. 从选项列表中选择“App IDs”(App ID)，然后点按“Continue”(继续)。

   ![image-20200418133907764](../.gitbook/assets/image-20200418133907764.png)

3. 在“Description”(描述) 栏位中输入 App ID 的名称或描述。

4. 在 Bundle ID 处选择“Explicit”并在输入框中 输入 app 的 Bundle ID。

   您在这里输入的 App ID 应该与 Xcode 中目标的“Summary”(摘要) 面板中输入的 Bundle ID 一致。

5. 选中相应的复选框，以启用您想要使用的 app 功能。

   “Capabilities”(功能) 下面会显示您的 app 类型和计划会员资格可以使用的功能。如果某个技术需要精确 App ID，而您创建的是通配符 App ID，或者该技术默认处于启用状态，则对应复选框会处于停用状态。并非所有功能都适用于所有平台。

   ![image-20200418134536338](../.gitbook/assets/image-20200418134536338.png)

6. 点按“Continue”(继续)，检查注册信息，然后点按“Register”(注册)。

   ![image-20200418134636139](../.gitbook/assets/image-20200418134636139.png)

   ![image-20200418134821694](../.gitbook/assets/image-20200418134821694.png)

## 创建 App Store 预置描述文件

您可以创建自己的 App Store 预置描述文件，在将 app 上传到 App Store Connect 时使用。

1. 在“[Certificates, Identifiers & Profiles (英文)](https://developer.apple.com/account/resources)”(证书、标识符和描述文件) 中，从侧边栏中选择“Profiles”(描述文件)，然后点按左上角的添加按钮 (+)。

   ![image-20200418135126942](../.gitbook/assets/image-20200418135126942.png)

2. 在“Distribution”(分发) 下面，选择“App Store” ，然后点按“Continue”(继续)。

   ![image-20200418135346290](../.gitbook/assets/image-20200418135346290.png)s

3. 从“App ID”弹出式菜单中选取用于开发的 App ID (与您的套装 ID 相匹配的 App ID)，然后点按“Continue”(继续)。

   ![image-20200418135514242](../.gitbook/assets/image-20200418135514242.png)

4. 选择您的分发证书，然后点按“Continue”(继续)。

   ![image-20200418135634625](../.gitbook/assets/image-20200418135634625.png)

5. 输入描述文件名称，然后点按“Generate”(生成)。

   ![image-20200418135744106](../.gitbook/assets/image-20200418135744106.png)

6. 点按“Download”(下载)。

   ![image-20200418135828283](../.gitbook/assets/image-20200418135828283.png)

## 创建 App 应用程序

1. 打开“xcode”开发工具，点击“Create a new Xode Project”创建一个新应用

   ![image-20200418140311500](../.gitbook/assets/image-20200418140311500.png)

2. 在 Application 下选择“Single View App”,点击“Next”继续

   ![image-20200418140641409](../.gitbook/assets/image-20200418140641409.png)

3. 输入信息，点击“Next”继续

   ![image-20200418141024145](../.gitbook/assets/image-20200418141024145.png)

4. 选择应用保存路径，点击“Create”创建应用

   ![image-20200418141151850](../.gitbook/assets/image-20200418141151850.png)

5. 编辑代码，编写随机数工具

   ![image-20200418142307421](img\image-20200418142307421.png)

6. 修改 Bundle ID，与注册 App ID 时一致

   ![image-20200418142801589](img\image-20200418142801589.png)

7. 配置签名，取消“Automatically manage signing”，选择“Provisioning Profile”点击导入，选择“创建 App Store 预置描述文件”下载的文件

   ![image-20200418144517653](../.gitbook/assets/image-20200418144517653.png)

   ![image-20200418144605843](../.gitbook/assets/image-20200418144605843.png)

8. 配置图标

   ![image-20200419002403770](../.gitbook/assets/image-20200419002403770.png)

## 添加 App 至您的帐户

您必须先在 App Store Connect 帐户中创建一个 App 记录，然后才能将 App 的构建版本上传至 App Store Connect。

1. 登录[App Store Connect](https://appstoreconnect.apple.com/)，点按“我的 App”。

   ![image-20200419003010221](../.gitbook/assets/image-20200419003010221.png)

2. 在左上角，点按“添加”按钮（+）,请从弹出菜单中选择“新建 App”

   ![image-20200419003318393](../.gitbook/assets/image-20200419003318393.png)

3. 在“新建 App”对话框中，选择一个或多个平台，并输入 App 信息,点击创建

   ![image-20200419003517790](../.gitbook/assets/image-20200419003517790.png)

4. 配置 App 信息，点击存储

   ![image-20200419004111699](../.gitbook/assets/image-20200419004111699.png)

5. 配置价格和销售范围,点击存储

   ![image-20200419004402028](../.gitbook/assets/image-20200419004402028.png)

6. 配置版本信息，点击存储

   ![image-20200419010811763](../.gitbook/assets/image-20200419010811763.png)

   ![image-20200419010921345](../.gitbook/assets/image-20200419010921345.png)

   ![image-20200419011043908](../.gitbook/assets/image-20200419011043908.png)

## 上传构建版本

1. 选择“Generic iOS Device”构建

   ![image-20200419011445487](../.gitbook/assets/image-20200419011445487.png)

2. 点击“Product”->“Archive”

   ![image-20200419011603852](../.gitbook/assets/image-20200419011603852.png)

3. 发布应用程序

   点击“Distribute App”

   ![image-20200419011804410](../.gitbook/assets/image-20200419011804410.png)

   选择“App Store Connect”，点击“Next”

   ![image-20200419011842235](../.gitbook/assets/image-20200419011842235.png)

   选择“Upload”，点击“Next”

   ![image-20200419011915107](../.gitbook/assets/image-20200419011915107.png)

   如果无法连接 “App Store Connect”，请点击“Manage Accounts”登录 Apple ID

   ![image-20200419012410584](../.gitbook/assets/image-20200419012410584.png)

   ![image-20200419012458013](../.gitbook/assets/image-20200419012458013.png)

   ![image-20200419012525943](../.gitbook/assets/image-20200419012525943.png)

   如果能连接“App Store Connect ”，点击“Next”

   ![image-20200419012944460](../.gitbook/assets/image-20200419012944460.png)

   选择“发放证书”和“App Store”预置描述文件，点击“Next”

   ![image-20200419013040498](../.gitbook/assets/image-20200419013040498.png)

   点击“Upload”

   ![image-20200419013154626](../.gitbook/assets/image-20200419013154626.png)

   上传成功，点击 Done

   ![image-20200419015321970](../.gitbook/assets/image-20200419015321970.png)

## 提交审核

1.  选择构建版本,点击存储

![image-20200419015552533](../.gitbook/assets/image-20200419015552533.png)

![image-20200419015627470](../.gitbook/assets/image-20200419015627470.png)

![image-20200419015722758](../.gitbook/assets/image-20200419015722758.png)

2. 提交以供审核

   ![image-20200419015845803](../.gitbook/assets/image-20200419015845803.png)

![image-20200419020030217](../.gitbook/assets/image-20200419020030217.png)
