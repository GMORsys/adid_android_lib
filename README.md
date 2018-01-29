# adid_android_lib

各AndroidのAdIDをGMOリサーチのサーバに送信するAndroidライブラリ。

## build.gradle の記載例

build.gradle に記載する。

```gradle

repositories {
    maven { url 'http://GMORsys.github.com/adid_android_lib/repository/' }
}

dependencies {
    implementation 'jp.gmo-research.adid:gmor-adid:1.3.1'
}

```

サンプルプログラム
view 側でボタンをクリックすると通信するアプリを想定

```java
// import は省略

public class MainActivity extends AppCompatActivity
        implements View.OnClickListener {

    private String advertisingId;

    /**
     * 広告ID を取得
     */
    class AdIdTask extends AsyncTask<Void, Void, String> {

        private Activity mActivity;

        AdIdTask(Activity activity) {
            mActivity = activity;
        }

        @Override
        protected String doInBackground(Void... params) {
            String advertisingId = new String();
            try {
                AdvertisingIdClient.Info info =
                        AdvertisingIdClient.getAdvertisingIdInfo(mActivity.getApplicationContext());
                advertisingId = info.getId();
            } catch (GooglePlayServicesNotAvailableException e) {
                //
            } catch (GooglePlayServicesRepairableException e) {
                //
            } catch (IOException e) {
                //
            }
            return advertisingId;
        }

        @Override
        protected void onPostExecute(String id) {
            advertisingId = id;
        }
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.button_device_regist).setOnClickListener(this);

        // 広告id 取得
        new AdIdTask(this).execute();
    }

    public void onClick(View v) {
    	// 下記の7つのパラメーターを渡す
        String monitorId = "001"; // モニタID (String)
        String panelType = "002"; // パネルタイプ (String)
        String deviceToken = ""; // デバイストークン 空固定にしておく (String) 
        // Android ID (String)
        String androidId = Settings.Secure.getString(getContentResolver(), Settings.System.ANDROID_ID);
        String adId = this.advertisingId; // 広告ID (String)
        String appId = "3"; // アプリ別のID。"3"固定。 (String)

        // アクセスする先の切り替え ST環境か本番環境か (Boolean)
        // TRUE : 本番環境
        // FALSE : ST環境
        Boolean urlFlag = Boolean.FALSE;

        Object[] postParamsMobileDeviceApi = new Object[7];
        postParamsMobileDeviceApi[0] = monitorId;
        postParamsMobileDeviceApi[1] = panelType;
        postParamsMobileDeviceApi[2] = deviceToken;
        postParamsMobileDeviceApi[3] = androidId;
        postParamsMobileDeviceApi[4] = adId;
        postParamsMobileDeviceApi[5] = appId;
        postParamsMobileDeviceApi[6] = urlFlag;

        new DeviceRegistAsyncTask(this).execute(postParamsMobileDeviceApi);
    }
}
```