# 使用AWS SSO（IAM Identity Center）通过CLI方式访问CodeCommit

## 配置IAM以及SSO

### 配置IAM相关权限

假设使用CodeCommit的用户需要两种不同的角色，管理员角色以及开发者角色，则需要创建这些角色所使用的IAM policy，可以根据需求创建不同的策略进行更精细的控制，本文仅以管理员和开发者为例（请参考source目录中json格式的IAM policy文件）。如果策略已经创建，请跳过此步骤。

1. 登录IAM console -> Policies，选择create policy：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image1.png?raw=true)

2. 根据步骤提示分别创建上述policy，并记下每个policy的名称

### 启用并配置AWS SSO（IAM Identity Center）

#### 为不同用户类似创建不同的permission set

1. 进入IAM Identity Center console界面，如果未启用IAM Identity Center，请先启用服务：https://docs.aws.amazon.com/singlesignon/latest/userguide/get-started-enable-identity-center.html，需要注意的是，对每个AWS账号来说，IAM Identity Center同时只能在同一个region启用，建议选择离用户比较近的region或CodeCommit服务所在的region

2. 启用IAM Identity Center后，进入Multi-account permissions -> Permission sets，选择create permission set，并选择Custom permission set，选择Next：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image2.png?raw=true)

3. 以创建CodeCommitPowerPolicy这个CodeCommit管理员的permission set为例，在Specify policies and permissions boundary界面，选择第二个选项Customer managed policies，并填入上一节中创建的相应的IAM policy名称，选择Next：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image3.png?raw=true)

4. 进入Permission set details页面，填入名称，选择duration时间（SSO登录用户有效期），relay state选项默认留空，选择Next，并最终选择Create。如需创建更多的permission set（如CodeCommit开发人员），请重复以上2-4步。最终创建成功的permission set会出现在如下界面列表当中：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image4.png?raw=true)

#### 为不同用户类型创建不同用户组

基于上一节中创建的permission set，可以创建相应的用户分组，后续加入新的用户，直接加入到相应权限的组内即可拥有相应的权限，无需额外配置

1. 进入IAM Identity Center console界面，选择Groups -> Create Group：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image5.png?raw=true)

2. 以创建管理员用户组为例，填入组名称，如CodeCommitPowerGroup，并选择Crate Group

3. 用户组创建好之后，为用户组分配相应的permission set。管理员用户组为例，进入IAM Identity Center console界面，选择AWS Accounts，右侧Organizational structure下，选择SSO用户组所在的账号，选择Assign users or groups：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image6.png?raw=true)

4. 在Assign users and groups界面，选择Groups，并选择上一步中新创建的用户组CodeCommitPowerGroup，选择Next：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image7.png?raw=true)

5. 在Assign permission sets界面，选择相应的permission set，选择Next，最后选择submit。对于其他用户组，请重复上述步骤进行操作

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image8.png?raw=true)

#### 为用户组分配用户

1. 进入IAM Identity Center console界面，以创建管理员用户为例，选择Users，Add user：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image9.png?raw=true)

2. 填入用户相关信息后，选择Next：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image10.png?raw=true)

3. 在Add user to groups在Add user to groups界面，选择新创建用户需要加入的用户组，选择Next，并最终选择Add user界面，选择新创建用户需要加入的用户组，选择Next，并最终选择Add user

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image11.png?raw=true)

4. 进入注册邮箱，进入注册邀请邮件，选择接受邀请激活用户，此时会打开新页面让用户设置登录密码，设置完成后即完成用户激活

5. 对于新加用户，可重复上述步骤

### 在AWS CLI上启用AWS SSO（IAM Identity Center）

本文以Mac OS环境为例，在AWS CLI上启用AWS SSO。本节需要用到上一节中SSO登录地址，可以在IAM Identity Center console界面，Settings->Identity source，找到AWS access portal URL：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image12.png?raw=true)

#### 前提条件

需要AWS CLI version 2，旧版本不支持AWS SSO。安装升级过程请参考https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html，如果已经安装AWS CLI version 1，需要先卸载再安装，请参考https://docs.aws.amazon.com/cli/latest/userguide/cliv2-migration-instructions.html

#### 在AWS CLI上配置AWS SSO

1. 打开终端软件，输入以下AWS CLI命令行，并输入上述步骤中的SSO AWS access portal URL，以及SSO的region信息：

```
aws configure sso
SSO start URL [None]: https://[domain].awsapps.com/start
SSO Region [None]: us-east-1
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:
https://device.sso.us-east-1.amazonaws.com/
Then enter the code:
XXXX-XXXX
```

2. 以上过程会自动打开浏览器进行操作，需要输入之前创建的用户名以及相应的密码，登录成功后在如下界面选择Allow即可返回CLI界面：

![alt text](https://github.com/zhixueli/sso-codecommit/blob/main/images/image13.png?raw=true)

3. 返回CLI界面继续操作，需要选择region信息以及CLI输出格式，默认即可，CLI profile name需要输入操作CodeCommit时想要使用的profile名称，以管理员为例，输入code-admin，管理员用户在使用CLI操作CodeCommit时（如同步代码，提交变更等）。如果需要查看或者更改profile的内容，可以通过再次运行命令行，或修改~/.aws/config文件：

```
The only role available to you is: CodeCommitPowerPolicy
Using the role name "CodeCommitPowerPolicy"
CLI default client Region [us-east-1]:
CLI default output format [json]:
CLI profile name [CodeCommitPowerPolicy-123456789012]: code-admin
To use this profile, specify the profile name using --profile, as shown:
aws s3 ls --profile code-admin
```

```
cat ~/.aws/config
[profile code-admin]
sso_start_url = https://[domain].awsapps.com/start
sso_region = us-east-1
sso_account_id = 123456789012
sso_role_name = CodeCommitPowerPolicy
region = us-east-1
output = json
```

## CodeCommit用户使用指南

### 前提条件

安装git-remote-codecommit插件：

```
pip install git-remote-codecommit
```

### 通过AWS CLI登录SSO

1. 在终端软件运行如下命令行，其中[codecommit user profile name]需要替换为已在AWS SSO创建的使用CodeCommit的profile，以管理员为例，替换为code-admin。命令行会自动打开浏览器访问SSO界面，并提示输入用户名和密码，登录成功后请选择Allow，回到CLI界面可以看到登录成功信息：

```
aws sso login --profile [codecommit user profile name]
Attempting to automatically open the SSO authorization page in your default browser.
If the browser does not open or you wish to use a different device to authorize this request, open the following URL:
https://device.sso.us-east-1.amazonaws.com/
Then enter the code:
XXXX-XXXX
Successfully logged into Start URL: https://[domain].awsapps.com/start
```

2. 此时终端已成功登录拥有相应的操作CodeCommit权限，可以通过Git命令对CodeCommit的repo进行操作。首次clone代码请使用如下命令，其它Git命令没有区别

```
# clone
git clone codecommit://[codecommit user profile name]@[codecommit repo name]
# sample
git clone codecommit://code-admin@company-repo
```

3. 代码操作完成后，如需登出SSO，请输入如下命令行。SSO登录之后获得的临时身份有过期时间，默认为一小时，管理员可以更改过期时间，过期后需要重新登录

```
aws sso logout
Successfully signed out of all SSO profiles.
```