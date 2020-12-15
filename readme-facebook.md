



今回ではFacebookでログインできるように `cordova-plugin-facebook4`を使います。  
`APP_ID`と`APP_NAME`は`cordova-plugin-facebook4`に必要なものだから、これからそれらを取得する方法を説明します。

1. [Facebook for Developers](https://developers.facebook.com/)にログインします。

2. `My Apps → Create App`でアプリを作成します。

 ![images-1](readme-img/facebook/images-1.png)

3. 必須な情報を入力したら、`Create App ID`をクリックします。

4. ダッシュボードの<i>Settings</i>から`APP_ID` (App ID) と `APP_NAME` を確認できます。
※　下で設定する必要があるから、どこかに保存してください。
![images-2](readme-img/facebook/images-2.png)

5. `+ Add Platform` でプラットフォームを選択します。

6. Androidで実行する場合、`Android`を選択します。
![images-3](readme-img/facebook/images-3.png)
* 以下の情報を入力したら、`Save Changes` を押してください。
    * `Google Play Package Name`: Android App SettingのMonaca Cloud IDEの設定でのAndroid’s Package Nameです。
    * `Key Hashes`: Android KeyStore SettingsのMonaca Cloud IDE で設定したKeyStoreのSHA-1 fingerprintです。SHA-1 fingerprintの取得方法は[こちら]((https://docs.monaca.io/en/faq/application/#how-to-get-sha-1-fingerprint-of-a-keystore-created-in-monaca-cloud-ide))でご参考ください。
    * `Single Sign On`を有効にします。
    ![images-5](readme-img/facebook/images-5.png)
7. iOS端末で実行する場合、 `+ Add Platform`をクリックしたら`iOS`を選択します。
    * 以下の情報を入力しますし、`Save Changes`で保存します。
        * `Bundle ID`: iOS App SettingsのMonaca Cloud IDEで設定したiOSのApp IDです。
        * `Single Sign On`を有効にします。
    ![images-4](readme-img/facebook/images-4.png)

### （①Facebook）管理画面上の設定
1. `App Settings`のところで`Social Login`を開きます
2. `Allow to login with facebok`で`allow`にチェックを入れますし、 `APP_ID` を入力します。
![images-9](readme-img/facebook/images-9.png)

## （①Facebook）Monacaの実装コード
### （①Facebook）Monacaプラグインの設定

1. Monacaを開きます
2. 上記のメニューバーから`Configure → Cordova Plugin Settings`をキリックします。
3. `Enbaled plugins`のところで、 `cordova-plugin-facebook4`の`Configure`ボタンをクリックします。
![images-7](readme-img/facebook/images-7.png)
4. 上で取得できた`APP_ID` と `APP_NAME`を入れます。
![images-8](readme-img/facebook/images-8.png)
5. `OK`ボタンで保存します。

## （①Facebook）コード説明
1. Componentsファイル

| File | Description |
| --- | --- |
| `index.html` | Login Page |
| `js/index.js` | 	A JavaScript file for implementation of the application |
| `js/service.js` | 	A JavaScript file for handle login  |

![images-9](readme-img/twitter/images-9.png)

2. `config.xml`
```xml
<platform name="android">
    <preference name="android-minSdkVersion" value="22"/>
    ...
</platform>
```

3. `index.html`
```html
<a href="#" id="FacebookLoginBtn" data-role="button" data-inline="false" data-theme="b">Login with Facebook</a>
```
4. `js/index.js`
```javascript
//------ Login with Facebook ------//
var LoginSuccess = function(response) {
    onFacebookLogin(response);    
};

var LoginError = function(error) {
    console.log('Login Error: ' + JSON.stringify(error));
    alert('Error retrieving user profile' + JSON.stringify(error));
};

function onFacebookLoginBtn() {
    facebookConnectPlugin.login(['email', 'public_profile'], LoginSuccess, LoginError);
}
```

5. `js/service.js`
```javascript
/**
 * This const is your NCMB key
 */
const ncmbproperty = {
    application_key : "YOUR_APPLICATION_KEY",
    client_key: "YOUR_CLIENT_KEY"
};

var ncmb = new NCMB(ncmbproperty.application_key, ncmbproperty.client_key);

//------ Login with Facebook ------//
function onFacebookLogin(response) {
    if (response.status === 'connected') {
        var auth = response.authResponse;
        var expire_date = new Date(  auth.expiresIn * 1000 + (new Date()).getTime()).toJSON();
        var date = {__type:"Date", iso:expire_date};
        var authData = {id:auth.userID,
                        access_token:auth.accessToken,
                        expiration_date:date};
        var user = new ncmb.User();
        user.signUpWith("facebook", authData)
            .then(function(user){
                return ncmb.User.loginWith(user);
            })
            .then(function(user){
                currentLoginUser = ncmb.User.getCurrentUser();
                alert("User Info: " + JSON.stringify(user));
            })
            .catch(function(err){
                alert("エラー" + JSON.stringify(err));
            });
    } else if (response.status === 'not_authorized') {
        alert(response.status);
    } else {
        alert('Please login to facebook!');
    }
}
```
