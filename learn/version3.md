### BeBop框架1.3

- 在version2的基础上优化,添加新的板块,构建比较完善的app

#### 1. 个人板块(包括了登陆注册)

创建`activity_personal.xml`页面,预设头像修改、昵称编辑、年龄设置、设置、退出账号等

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <androidx.appcompat.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
        android:popupTheme="@style/ThemeOverlay.AppCompat.Light" />
    <!-- 示例内容，可以根据实际需要修改 -->
    <!-- 用户头像 -->
    <ImageView
        android:id="@+id/user_avatar"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_centerHorizontal="true"
        android:layout_marginTop="16dp"
        android:src="@drawable/ic_logo"
        android:contentDescription="@string/user_avatar" />
    <Button
        android:id="@+id/buttonChangeAvatar"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Change Avatar" />
    <!-- 昵称编辑 -->
    <EditText
        android:id="@+id/nickname_edit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/user_avatar"
        android:layout_marginTop="16dp"
        android:hint="@string/nickname_hint"
        android:inputType="textPersonName" />
    <!-- 年龄编辑 -->
    <EditText
        android:id="@+id/age_edit"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/nickname_edit"
        android:layout_marginTop="16dp"
        android:hint="@string/age_hint"
        android:inputType="number" />

    <!-- 设置按钮 -->
    <Button
        android:id="@+id/settings_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/age_edit"
        android:layout_marginTop="16dp"
        android:text="@string/settings_button_text" />

    <!-- 退出登录按钮 -->
    <Button
        android:id="@+id/logout_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/settings_button"
        android:layout_marginTop="16dp"
        android:text="@string/logout_button_text"
        android:backgroundTint="@android:color/holo_red_dark"
        android:textColor="@android:color/white" />
</LinearLayout>
```

#### 2. 创建`custom_dialog_title.xml`登陆注册页面的标题和退出按键

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="10dp">

    <TextView
        android:id="@+id/dialog_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Login or Register"
        android:textSize="18sp"
        android:textStyle="bold"
        android:layout_centerInParent="true" />

    <ImageButton
        android:id="@+id/dialog_close_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_menu_close_clear_cancel"
        android:background="?attr/selectableItemBackground"
        android:layout_alignParentEnd="true"
        android:layout_centerVertical="true" />
</RelativeLayout>
```

#### 3. 创建`register_dialog.xml`登陆注册页面,需要设置Tab切换按键实现登陆与注册的切换

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <!-- Tab 切换按钮 -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:id="@+id/login_tab"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@string/login_tab_text" />

        <Button
            android:id="@+id/register_tab"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="@string/register_tab_text" />
    </LinearLayout>

    <!-- 登录板块 -->
    <LinearLayout
        android:id="@+id/login_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_below="@id/login_tab"
        android:visibility="visible">

        <EditText
            android:id="@+id/login_username"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/username_hint"
            android:inputType="text" />

        <EditText
            android:id="@+id/login_password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/password_hint"
            android:inputType="textPassword" />

        <Button
            android:id="@+id/login_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/login_button_text" />
    </LinearLayout>

    <!-- 注册板块 -->
    <LinearLayout
        android:id="@+id/register_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:layout_below="@id/login_tab"
        android:visibility="gone">

        <EditText
            android:id="@+id/register_username"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/username_hint"
            android:inputType="text" />

        <EditText
            android:id="@+id/register_password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/password_hint"
            android:inputType="textPassword" />

        <EditText
            android:id="@+id/register_confirm_password"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:hint="@string/confirm_password_hint"
            android:inputType="textPassword" />

        <Button
            android:id="@+id/register_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="@string/register_button_text" />
    </LinearLayout>
</LinearLayout>
```

#### 4. 创建`PersonalActivity.java`实现页面逻辑

```java
public class PersonalActivity extends AppCompatActivity {

    private static final int PICK_IMAGE_REQUEST = 1;
    private ImageView imageViewAvatar;
    private SharedPreferences sharedPreferences;
    private static final String PREFS_NAME = "UserPrefs";
    private static final String IS_LOGGED_IN = "IsLoggedIn";
    private static final String AVATAR_KEY = "avatar";
    private DatabaseHelper db;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_personal);

        // 设置标题
        setTitle("Personal");

        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);
        if (getSupportActionBar() != null) {
            getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        }

        sharedPreferences = getSharedPreferences(PREFS_NAME, MODE_PRIVATE);

        if (!isUserLoggedIn()) {
            showRegisterDialog();
        }

        // 设置返回键功能
        getSupportActionBar().setDisplayHomeAsUpEnabled(true);

        Button logoutButton = findViewById(R.id.logout_button);
        logoutButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                sharedPreferences.edit().putBoolean(IS_LOGGED_IN, false).apply();
                showRegisterDialog();
            }
        });


        imageViewAvatar = findViewById(R.id.user_avatar);
        sharedPreferences = getSharedPreferences(PREFS_NAME, MODE_PRIVATE);

        // 加载保存的头像
        String avatarPath = sharedPreferences.getString(AVATAR_KEY, null);
        if (avatarPath != null) {
            imageViewAvatar.setImageURI(Uri.parse(avatarPath));
        }

        Button buttonChangeAvatar = findViewById(R.id.buttonChangeAvatar);
        buttonChangeAvatar.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 打开相册选择图片
                Intent intent = new Intent();
                intent.setType("image/*");
                intent.setAction(Intent.ACTION_GET_CONTENT);
                startActivityForResult(Intent.createChooser(intent, "Select Picture"), PICK_IMAGE_REQUEST);
            }
        });

        if (ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.READ_EXTERNAL_STORAGE}, 1);
        }
        db = new DatabaseHelper(this);
    }

    private boolean isUserLoggedIn() {
        return sharedPreferences.getBoolean(IS_LOGGED_IN, false);
    }

    private void showRegisterDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(this);
        LayoutInflater inflater = this.getLayoutInflater();
        View dialogView = inflater.inflate(R.layout.register_dialog, null);
        View customTitleView = inflater.inflate(R.layout.custom_dialog_title, null);

        builder.setView(dialogView);
        builder.setCustomTitle(customTitleView);

        AlertDialog dialog = builder.create();
        Button loginTab = dialogView.findViewById(R.id.login_tab);
        Button registerTab = dialogView.findViewById(R.id.register_tab);
        LinearLayout loginLayout = dialogView.findViewById(R.id.login_layout);
        LinearLayout registerLayout = dialogView.findViewById(R.id.register_layout);

        loginTab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loginLayout.setVisibility(View.VISIBLE);
                registerLayout.setVisibility(View.GONE);
            }
        });

        registerTab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                loginLayout.setVisibility(View.GONE);
                registerLayout.setVisibility(View.VISIBLE);
            }
        });

        // 在自定义标题视图中设置关闭按钮的监听器
        ImageButton closeButton = customTitleView.findViewById(R.id.dialog_close_button);
        closeButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dialog.dismiss();
            }
        });

        // 登录逻辑
        EditText loginUsername = dialogView.findViewById(R.id.login_username);
        EditText loginPassword = dialogView.findViewById(R.id.login_password);
        Button loginButton = dialogView.findViewById(R.id.login_button);

        loginButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String username = loginUsername.getText().toString();
                String password = loginPassword.getText().toString();
                // 在这里添加实际的登录逻辑
                if (db.checkUser(username, password)) {
                    sharedPreferences.edit().putBoolean(IS_LOGGED_IN, true).apply();
                    Toast.makeText(PersonalActivity.this, "登录成功", Toast.LENGTH_SHORT).show();
                    dialog.dismiss();
                } else {
                    Toast.makeText(PersonalActivity.this, "登录失败", Toast.LENGTH_SHORT).show();
                }
            }
        });

        // 注册逻辑
        EditText registerUsername = dialogView.findViewById(R.id.register_username);
        EditText registerPassword = dialogView.findViewById(R.id.register_password);
        EditText registerConfirmPassword = dialogView.findViewById(R.id.register_confirm_password);
        Button registerButton = dialogView.findViewById(R.id.register_button);

        registerButton.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String username = registerUsername.getText().toString();
                String password = registerPassword.getText().toString();
                String confirmPassword = registerConfirmPassword.getText().toString();

                if (password.equals(confirmPassword)) {
                    // 检查用户名和密码是否为空
                    if (!username.isEmpty() && !password.isEmpty()) {
                        // 检查用户名是否已经存在
                        if (db.checkUser(username)) {
                            Toast.makeText(PersonalActivity.this, "用户名已存在", Toast.LENGTH_SHORT).show();
                        } else {
                            // 注册用户
                            db.addUser(username, password);
                            sharedPreferences.edit().putBoolean(IS_LOGGED_IN, true).apply();
                            Toast.makeText(PersonalActivity.this, "注册成功", Toast.LENGTH_SHORT).show();
                            dialog.dismiss();
                        }
                    } else {
                        Toast.makeText(PersonalActivity.this, "用户名或密码不能为空", Toast.LENGTH_SHORT).show();
                    }
                } else {
                    Toast.makeText(PersonalActivity.this, "两次密码输入不一致", Toast.LENGTH_SHORT).show();
                }
            }
        });
        dialog.show();
    }


    @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        if (requestCode == PICK_IMAGE_REQUEST && resultCode == RESULT_OK && data != null && data.getData() != null) {
            Uri selectedImageUri = data.getData();
            imageViewAvatar.setImageURI(selectedImageUri);

            // 保存头像路径
            SharedPreferences.Editor editor = sharedPreferences.edit();
            editor.putString(AVATAR_KEY, selectedImageUri.toString());
            editor.apply();
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (requestCode == 1) {
            if (grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                // Permission granted
            } else {
                // Permission denied
                //Toast.makeText(this, "Permission denied to read your External storage", Toast.LENGTH_SHORT).show();
            }
        }
    }
    @Override
    public boolean onSupportNavigateUp() {
        finish();
        return true;
    }
}
```

#### 5. 开屏动画

创建`splash_screen.xml`文件,使用ImageView组件确定动画的位置

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/white">

    <ImageView
        android:id="@+id/splash_animation"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true" />
</RelativeLayout>
```

#### 6. 在res\drawable中创建`splash_animation.xml`文件,将每一帧图片导入

```xml
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="true">

    <item
        android:drawable="@drawable/animation_fufu1"
        android:duration="200" />
    <item
        android:drawable="@drawable/animation_fufu2"
        android:duration="200" />
    <item
        android:drawable="@drawable/animation_fufu3"
        android:duration="200" />
    <!-- 添加所有帧 -->
</animation-list>
```
#### 7. 在res\drawable中创建`splash_screen_background.xml`文件,调整动画背景

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@color/white" />
    <item>
        <bitmap
            android:gravity="center"
            android:src="@color/white" />
    </item>
</layer-list>
```

#### 8. 创建`SplashActivity.java`文件,实现动画效果

```java
public class SplashActivity extends AppCompatActivity {

    private AnimationDrawable splashAnimation;
    private static final String TAG = "SplashActivity";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.splash_screen);

        final ImageView splashImageView = findViewById(R.id.splash_animation);
        splashImageView.setBackgroundResource(R.drawable.splash_animation);

        splashImageView.post(new Runnable() {
            @Override
            public void run() {
                splashAnimation = (AnimationDrawable) splashImageView.getBackground();
                if (splashAnimation == null) {
                    Log.e(TAG, "splashAnimation is null");
                    return;
                }
                splashAnimation.start();

                // 计算动画的持续时间
                int duration = getAnimationDuration();

                // 使用动画的持续时间进行跳转
                new Handler().postDelayed(new Runnable() {
                    @Override
                    public void run() {
                        Intent intent = new Intent(SplashActivity.this, MainActivity.class);
                        startActivity(intent);
                        finish();
                    }
                }, duration);
            }
        });
    }

    private int getAnimationDuration() {
        if (splashAnimation == null) {
            Log.e(TAG, "splashAnimation is null in getAnimationDuration");
            return 0;
        }
        int duration = 0;
        for (int i = 0; i < splashAnimation.getNumberOfFrames(); i++) {
            duration += splashAnimation.getDuration(i);
        }
        return duration;
    }
}
```
#### 9. 调整`AndroidManifest.xml`文件,将`SplashActivity`设置为启动页面

```xml
<activity android:name=".SplashActivity"
    android:theme="@style/SplashTheme"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

#### 10. UI美化,图标美化,页面标题调整

- 导入图片调整侧边导航栏`select_menu.xml`图标

- 调整出现的标题,如Home页面的标题改为pp

```java
public class HomeFragment extends Fragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_home, container, false);

        // 设置标题
        getActivity().setTitle("pp");

        return view;
    }
}
```