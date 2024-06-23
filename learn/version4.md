### BeBop框架1.4

- 使用`Gallery`页面,搭建学生管理系统

#### 1. 数据库sqlite的创建

- 创建`DatabaseHelper.java`文件

```java
public class DatabaseHelper extends SQLiteOpenHelper {
    private static final String DATABASE_NAME = "user_db";
    private static final int DATABASE_VERSION = 2;

    private static final String TABLE_USERS = "users";
    private static final String COLUMN_ID = "id";
    private static final String COLUMN_USERNAME = "username";
    private static final String COLUMN_PASSWORD = "password";
    // 学生信息表
    private static final String TABLE_STUDENTS = "students";
    private static final String STUDENT_ID = "student_id";
    private static final String STUDENT_NAME = "student_name";
    private static final String STUDENT_GENDER = "student_gender";
    private static final String STUDENT_AGE = "student_age";
    private static final String STUDENT_BIRTHDATE = "student_birthdate";
    private static final String STUDENT_ADDRESS = "student_address";
    private static final String STUDENT_PHONE = "student_phone";

    private static final String TABLE_CREATE =
            "CREATE TABLE " + TABLE_USERS + " (" +
                    COLUMN_ID + " INTEGER PRIMARY KEY AUTOINCREMENT, " +
                    COLUMN_USERNAME + " TEXT, " +
                    COLUMN_PASSWORD + " TEXT);";
    // 创建学生信息表的 SQL 语句
    private static final String TABLE_CREATE_STUDENTS =
            "CREATE TABLE " + TABLE_STUDENTS + " (" +
                    STUDENT_ID + " INTEGER PRIMARY KEY, " +
                    STUDENT_NAME + " TEXT, " +
                    STUDENT_GENDER + " TEXT, " +
                    STUDENT_AGE + " INTEGER, " +
                    STUDENT_BIRTHDATE + " TEXT, " +
                    STUDENT_ADDRESS + " TEXT, " +
                    STUDENT_PHONE + " TEXT);";
    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(TABLE_CREATE);
        db.execSQL(TABLE_CREATE_STUDENTS);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_USERS);
        db.execSQL("DROP TABLE IF EXISTS " + TABLE_STUDENTS);
        onCreate(db);
    }
    public void addUser(String username, String password) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(COLUMN_USERNAME, username);
        values.put(COLUMN_PASSWORD, password);

        db.insert(TABLE_USERS, null, values);
        db.close();
    }
    public boolean checkUser(String username) {
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.query(TABLE_USERS, new String[]{COLUMN_USERNAME},
                COLUMN_USERNAME + "=?", new String[]{username}, null, null, null);
        int count = cursor.getCount();
        cursor.close();
        db.close();
        return count > 0;
    }
    public boolean checkUser(String username, String password) {
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.query(TABLE_USERS, new String[]{COLUMN_USERNAME},
                COLUMN_USERNAME + "=? AND " + COLUMN_PASSWORD + "=?", new String[]{username, password}, null, null, null);
        int count = cursor.getCount();
        cursor.close();
        db.close();
        return count > 0;
    }

    public void addStudent(Student student) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(STUDENT_ID, student.getId());
        values.put(STUDENT_NAME, student.getName());
        values.put(STUDENT_GENDER, student.getGender());
        values.put(STUDENT_AGE, student.getAge());
        values.put(STUDENT_BIRTHDATE, student.getBirthdate());
        values.put(STUDENT_ADDRESS, student.getAddress());
        values.put(STUDENT_PHONE, student.getPhone());

        db.insert(TABLE_STUDENTS, null, values);
        db.close();
    }

    public List<Student> getAllStudents() {
        List<Student> students = new ArrayList<>();
        SQLiteDatabase db = null;
        Cursor cursor = null;
        try {
            db = this.getReadableDatabase();
            cursor = db.query(TABLE_STUDENTS, null, null, null, null, null, null);

            if (cursor != null && cursor.moveToFirst()) {
                do {
                    int id = cursor.getInt(cursor.getColumnIndexOrThrow(STUDENT_ID));
                    String name = cursor.getString(cursor.getColumnIndexOrThrow(STUDENT_NAME));
                    String gender = cursor.getString(cursor.getColumnIndexOrThrow(STUDENT_GENDER));
                    int age = cursor.getInt(cursor.getColumnIndexOrThrow(STUDENT_AGE));
                    String birthdate = cursor.getString(cursor.getColumnIndexOrThrow(STUDENT_BIRTHDATE));
                    String address = cursor.getString(cursor.getColumnIndexOrThrow(STUDENT_ADDRESS));
                    String phone = cursor.getString(cursor.getColumnIndexOrThrow(STUDENT_PHONE));

                    students.add(new Student(id, name, gender, age, birthdate, address, phone));
                } while (cursor.moveToNext());
            }
        } catch (Exception e) {
            // 处理异常，例如记录日志
            e.printStackTrace();
        } finally {
            if (cursor != null) {
                cursor.close();
            }
            if (db != null && db.isOpen()) {
                db.close();
            }
        }

        return students;
    }
    // 更新学生信息
    public int updateStudent(int oldStudentId, int newStudentId, String name, String gender, int age, String birthdate, String address, String phone) {
        SQLiteDatabase db = this.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put("student_id", newStudentId); // 新的 STUDENT_ID
        values.put(STUDENT_NAME, name);
        values.put(STUDENT_GENDER, gender);
        values.put(STUDENT_AGE, age);
        values.put(STUDENT_BIRTHDATE, birthdate);
        values.put(STUDENT_ADDRESS, address);
        values.put(STUDENT_PHONE, phone);

        // 使用STUDENT_ID进行条件匹配进行更新
        int rowsAffected = db.update(TABLE_STUDENTS, values, STUDENT_ID + " = ?", new String[]{String.valueOf(oldStudentId)});
        db.close();
        return rowsAffected;
    }
    // 删除学生信息
    public int deleteStudent(int studentId) {
        SQLiteDatabase db = this.getWritableDatabase();
        int rowsAffected = db.delete(TABLE_STUDENTS, STUDENT_ID + " = ?", new String[]{String.valueOf(studentId)});
        db.close();
        return rowsAffected;
    }
}
```

#### 2. 账号的登录与注册(用户使用`Gallery`页面,将会检测是否为登录状态)

```java
public class GalleryFragment extends Fragment {
    private static final String IS_LOGGED_IN = "IsLoggedIn";
    private static final String PREFS_NAME = "UserPrefs";
    private SharedPreferences sharedPreferences;
    private DatabaseHelper db;
    private RecyclerView recyclerView;
    private GalleryAdapter galleryAdapter;
    private List<Student> studentList = new ArrayList<>();// 初始化数据列表
    private List<Student> filteredStudentList = new ArrayList<>(); // 过滤后的学生列表

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_gallery, container, false);

        ImageView addIcon = view.findViewById(R.id.add_icon);
        recyclerView = view.findViewById(R.id.recycler_view);
        // 初始化RecyclerView
        recyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
        // 初始化时添加一些数据
        initializeData(view.getContext());
        setupRecyclerView(view);
        // 设置适配器
        recyclerView.setAdapter(galleryAdapter);

        // 设置添加图标的点击事件
        addIcon.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                showAddItemDialog();
            }
        });
        // 设置标题
        getActivity().setTitle("Gallery");

        return view;
    }

    private void addItem(Student student) {
        DatabaseHelper dbHelper = new DatabaseHelper(getContext());
        dbHelper.addStudent(student);
        studentList.clear();
        studentList.addAll(dbHelper.getAllStudents());
        galleryAdapter.notifyDataSetChanged();
        Toast.makeText(getContext(), "Item Added", Toast.LENGTH_SHORT).show();
    }
    private void initializeData(Context context) {
        DatabaseHelper dbHelper = new DatabaseHelper(getContext());
        studentList = dbHelper.getAllStudents();
        // 初始时将所有数据添加到过滤列表中
        filteredStudentList = new ArrayList<>(studentList);
        galleryAdapter = new GalleryAdapter(context,filteredStudentList);
    }
    private void setupRecyclerView(View view) {
        RecyclerView recyclerView = view.findViewById(R.id.recycler_view);
        recyclerView.setLayoutManager(new LinearLayoutManager(getContext()));
        recyclerView.setAdapter(galleryAdapter);
    }
    public void filterStudents(String query) {
        filteredStudentList.clear();
        if (query.isEmpty()) {
            filteredStudentList.addAll(studentList);
        } else {
            for (Student student : studentList) {
                String idString = String.valueOf(student.getId());
                String name = student.getName();
                if ((idString.contains(query)) || (name != null && name.contains(query))) {
                    filteredStudentList.add(0, student); // 将匹配的学生信息添加到列表最前面
                }
            }
        }
        galleryAdapter.notifyDataSetChanged();
    }
    @Override
    public void onResume() {
        super.onResume();
        // 初始化 DatabaseHelper
        db = new DatabaseHelper(getActivity());
        // 获取 SharedPreferences
        sharedPreferences = getActivity().getSharedPreferences(PREFS_NAME, Context.MODE_PRIVATE);

        if (!isUserLoggedIn()) {
            showRegisterDialog();
            getFragmentManager().beginTransaction()
                    .replace(R.id.content_frame, new HomeFragment())
                    .commit();
        }
    }

    private boolean isUserLoggedIn() { return sharedPreferences.getBoolean(IS_LOGGED_IN, false);}

    private void showRegisterDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(getActivity());
        LayoutInflater inflater = getLayoutInflater();
        View dialogView = inflater.inflate(R.layout.register_dialog, null);
        View customTitleView = inflater.inflate(R.layout.custom_dialog_title, null);

        builder.setView(dialogView);
        builder.setCustomTitle(customTitleView);

        AlertDialog dialog = builder.create();
        dialog.show();

        Button loginTab = dialogView.findViewById(R.id.login_tab);
        Button registerTab = dialogView.findViewById(R.id.register_tab);
        LinearLayout loginLayout = dialogView.findViewById(R.id.login_layout);
        LinearLayout registerLayout = dialogView.findViewById(R.id.register_layout);

        // 添加按钮点击事件处理逻辑
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
                    Toast.makeText(getActivity(), "登录成功", Toast.LENGTH_SHORT).show();
                    dialog.dismiss();
                } else {
                    Toast.makeText(getActivity(),"登录失败", Toast.LENGTH_SHORT).show();
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
                            Toast.makeText(getActivity(), "用户名已存在", Toast.LENGTH_SHORT).show();
                        } else {
                            // 注册用户
                            db.addUser(username, password);
                            sharedPreferences.edit().putBoolean(IS_LOGGED_IN, true).apply();
                            Toast.makeText(getActivity(), "注册成功", Toast.LENGTH_SHORT).show();
                            dialog.dismiss();
                        }
                    } else {
                        Toast.makeText(getActivity(), "用户名或密码不能为空", Toast.LENGTH_SHORT).show();
                    }
                } else {
                    Toast.makeText(getActivity(), "两次密码输入不一致", Toast.LENGTH_SHORT).show();
                }
            }
        });
        dialog.show();
    }

    private void showAddItemDialog() {
        AlertDialog.Builder builder = new AlertDialog.Builder(getContext());
        LayoutInflater inflater = getLayoutInflater();
        View dialogView = inflater.inflate(R.layout.addstudent_dialog, null);
        View customTitleView = inflater.inflate(R.layout.student_dialog_title, null);
        builder.setView(dialogView);

        builder.setCustomTitle(customTitleView);

        EditText editTextStudentId = dialogView.findViewById(R.id.student_id);
        EditText editTextStudentName = dialogView.findViewById(R.id.student_name);
        EditText editTextStudentGender = dialogView.findViewById(R.id.student_gender);
        EditText editTextStudentAge = dialogView.findViewById(R.id.student_age);
        EditText editTextStudentBirthdate = dialogView.findViewById(R.id.student_birthdate);
        EditText editTextStudentAddress = dialogView.findViewById(R.id.student_address);
        EditText editTextStudentPhone = dialogView.findViewById(R.id.student_phone);
        Button buttonConfirm = dialogView.findViewById(R.id.buttonConfirm);
        Button buttonCancel = dialogView.findViewById(R.id.buttonCancel);

        AlertDialog dialog = builder.create();

        buttonConfirm.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String studentIdStr = editTextStudentId.getText().toString();
                String studentName = editTextStudentName.getText().toString();
                String studentGender = editTextStudentGender.getText().toString();
                String studentAge = editTextStudentAge.getText().toString();
                String studentBirthdate = editTextStudentBirthdate.getText().toString();
                String studentAddress = editTextStudentAddress.getText().toString();
                String studentPhone = editTextStudentPhone.getText().toString();

                if (!studentIdStr.isEmpty() && !studentName.isEmpty()&& !studentGender.isEmpty() && !studentAge.isEmpty() && !studentBirthdate.isEmpty() && !studentAddress.isEmpty() && !studentPhone.isEmpty()) {
                    addItem(new Student(Integer.parseInt(studentIdStr), studentName, studentGender, Integer.parseInt(studentAge), studentBirthdate, studentAddress, studentPhone));
                    dialog.dismiss();
                } else {
                    Toast.makeText(getContext(), "请填写所有字段", Toast.LENGTH_SHORT).show();
                }
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

        buttonCancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dialog.dismiss();
            }
        });

        dialog.show();
    }
}
```
#### 3. 学生信息管理系统

- 设计`fragment_gallery.xml`页面,使用`ImageView`控件添加学生信息,使用`RecyclerView`控件呈现信息

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".GalleryFragment">

    <ImageView
        android:id="@+id/add_icon"
        android:layout_width="match_parent"
        android:layout_height="150dp"
        android:layout_margin="12dp"
        android:padding="16dp"
        android:src="@drawable/ic_add_r"
        android:contentDescription="@string/add_icon_description"/>
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:padding="16dp" />
</LinearLayout>
```
- 创建`item_gallery.xml`文件,呈现每个学生信息

```xml
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="@android:color/white"
    android:elevation="4dp"
    android:layout_margin="8dp">

    <TextView
        android:id="@+id/student_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="学号："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <TextView
        android:id="@+id/student_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="姓名："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <TextView
        android:id="@+id/student_gender"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="性别："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <TextView
        android:id="@+id/student_age"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="年龄："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <TextView
        android:id="@+id/student_birthdate"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="出生日期："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <TextView
        android:id="@+id/student_address"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="家庭住址："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <TextView
        android:id="@+id/student_phone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="电话号码："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>
    <LinearLayout
        android:id="@+id/buttons_container"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_alignParentLeft="true"
        android:layout_marginTop="8dp">

        <ImageView
            android:id="@+id/edit_icon"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:src="@drawable/ic_edit"
            android:contentDescription="@string/edit_icon_description"
            android:layout_marginRight="16dp"
            android:padding="8dp" />

        <ImageView
            android:id="@+id/delete_icon"
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:src="@drawable/ic_delete"
            android:contentDescription="@string/delete_icon_description"
            android:padding="8dp" />
    </LinearLayout>
</LinearLayout>
```

#### 4. 个人信息录入：学号、姓名、性别、年龄、家庭住址、电话号码

- 创建`addstudent_dialog.xml`文件,实现学生信息的录入

```xml
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="@android:color/white"
    android:elevation="4dp"
    android:layout_margin="8dp">

    <EditText
        android:id="@+id/student_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="学号："
        android:inputType="number"
        android:textSize="16sp"
        android:textColor="@android:color/black"/>
    <EditText
        android:id="@+id/student_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="姓名："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_gender"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="性别："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_age"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="年龄："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_birthdate"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="出生日期："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_address"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="家庭住址："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_phone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="电话号码："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>
    <Button
        android:id="@+id/buttonConfirm"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="确认" />

    <Button
        android:id="@+id/buttonCancel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="取消" />
</LinearLayout>
```

- 创建`GalleryAdapter.java`适配器文件

```java
public class GalleryAdapter extends RecyclerView.Adapter<GalleryAdapter.GalleryViewHolder> {

    private Context context;
    private List<Student> studentList;
    private DatabaseHelper dbHelper;
    private Activity activity;

    public GalleryAdapter(Context context,List<Student> studentList) {
        this.context = context;
        this.studentList = studentList;
        this.dbHelper = new DatabaseHelper(context);
        this.activity = activity;
    }

    @NonNull
    @Override
    public GalleryViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = LayoutInflater.from(context).inflate(R.layout.item_gallery, parent, false);
        return new GalleryViewHolder(view);
    }

    @Override
    public void onBindViewHolder(@NonNull GalleryViewHolder holder, int position) {
        Student student = studentList.get(position);
        holder.studentId.setText("学号："+String.valueOf(student.getId()));
        holder.studentName.setText("姓名："+student.getName());
        holder.studentGender.setText("性别："+student.getGender());
        holder.studentAge.setText("年龄："+String.valueOf(student.getAge()));
        holder.studentBirthdate.setText("出生日期："+student.getBirthdate());
        holder.studentAddress.setText("家庭住址："+student.getAddress());
        holder.studentPhone.setText("电话号码："+student.getPhone());
        holder.editIcon.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                // 处理编辑逻辑
                showUpdateDialog(student);
            }
        });
        holder.deleteIcon.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                int adapterPosition = holder.getAdapterPosition();
                // 处理删除逻辑
                int rowsAffected = dbHelper.deleteStudent(student.getId());
                if (rowsAffected > 0) {
                    studentList.remove(adapterPosition);
                    notifyItemRemoved(adapterPosition);
                    notifyItemRangeChanged(adapterPosition, studentList.size());
                    Toast.makeText(v.getContext(), "Deleted " + student.getName(), Toast.LENGTH_SHORT).show();
                } else {
                    Toast.makeText(v.getContext(), "Failed to delete " + student.getName(), Toast.LENGTH_SHORT).show();
                }
            }
        });
    }

    @Override
    public int getItemCount() {
        return studentList.size();
    }

    public static class GalleryViewHolder extends RecyclerView.ViewHolder {
        public TextView studentId;
        public TextView studentName;
        public TextView studentGender;
        public TextView studentAge;
        public TextView studentBirthdate;
        public TextView studentAddress;
        public TextView studentPhone;
        public ImageView editIcon;
        public ImageView deleteIcon;

        public GalleryViewHolder(View itemView) {
            super(itemView);
            studentId = itemView.findViewById(R.id.student_id);
            studentName = itemView.findViewById(R.id.student_name);
            studentGender = itemView.findViewById(R.id.student_gender);
            studentAge = itemView.findViewById(R.id.student_age);
            studentBirthdate = itemView.findViewById(R.id.student_birthdate);
            studentAddress = itemView.findViewById(R.id.student_address);
            studentPhone = itemView.findViewById(R.id.student_phone);
            editIcon = itemView.findViewById(R.id.edit_icon);
            deleteIcon = itemView.findViewById(R.id.delete_icon);
        }
    }
    private void showUpdateDialog(Student student) {
        // 创建并显示一个更新对话框
        AlertDialog.Builder builder = new AlertDialog.Builder(context);
        LayoutInflater inflater = LayoutInflater.from(context);
        View dialogView = inflater.inflate(R.layout.student_dialog, null);
        View customTitleView = inflater.inflate(R.layout.student_dialog_title, null);
        builder.setView(dialogView);

        builder.setCustomTitle(customTitleView);

        EditText editTextOldStudentId = dialogView.findViewById(R.id.student_old_id);
        EditText editTextNewStudentId = dialogView.findViewById(R.id.student_new_id);
        EditText editTextStudentName = dialogView.findViewById(R.id.student_name);
        EditText editTextStudentGender = dialogView.findViewById(R.id.student_gender);
        EditText editTextStudentAge = dialogView.findViewById(R.id.student_age);
        EditText editTextStudentBirthdate = dialogView.findViewById(R.id.student_birthdate);
        EditText editTextStudentAddress = dialogView.findViewById(R.id.student_address);
        EditText editTextStudentPhone = dialogView.findViewById(R.id.student_phone);
        Button buttonConfirm = dialogView.findViewById(R.id.buttonConfirm);
        Button buttonCancel = dialogView.findViewById(R.id.buttonCancel);

        AlertDialog dialog = builder.create();



        buttonConfirm.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String oldStudentIdStr = editTextOldStudentId.getText().toString();
                String newStudentIdStr = editTextNewStudentId.getText().toString();
                String studentName = editTextStudentName.getText().toString();
                String studentGender = editTextStudentGender.getText().toString();
                String studentAge = editTextStudentAge.getText().toString();
                String studentBirthdate = editTextStudentBirthdate.getText().toString();
                String studentAddress = editTextStudentAddress.getText().toString();
                String studentPhone = editTextStudentPhone.getText().toString();

                updateItem(oldStudentIdStr, newStudentIdStr, studentName, studentGender, studentAge, studentBirthdate, studentAddress, studentPhone);
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

        buttonCancel.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                dialog.dismiss();
            }
        });

        dialog.show();
    }
    private void updateItem(String oldStudentIdStr, String newStudentIdStr, String studentName, String studentGender, String studentAge, String studentBirthdate, String studentAddress, String studentPhone) {
        if (!oldStudentIdStr.isEmpty() && !studentName.isEmpty() && !studentGender.isEmpty() && !studentAge.isEmpty() && !studentBirthdate.isEmpty() && !studentAddress.isEmpty() && !studentPhone.isEmpty()) {
            try {
                int oldStudentId = Integer.parseInt(oldStudentIdStr);
                int newStudentId = Integer.parseInt(newStudentIdStr);
                int age = Integer.parseInt(studentAge);

                DatabaseHelper dbHelper = new DatabaseHelper(context);
                int rowsAffected = dbHelper.updateStudent(oldStudentId, newStudentId, studentName, studentGender, age, studentBirthdate, studentAddress, studentPhone);

                if (rowsAffected > 0) {
                    studentList.clear();
                    studentList.addAll(dbHelper.getAllStudents());
                    notifyDataSetChanged();
                    Toast.makeText(context, "学生信息更新成功", Toast.LENGTH_SHORT).show();
                    // 通知Activity关闭窗口
                    activity.finish();
                } else {
                    Toast.makeText(context, "学生信息更新失败", Toast.LENGTH_SHORT).show();
                }
            } catch (NumberFormatException e) {
                Toast.makeText(context, "学号或年龄格式不正确", Toast.LENGTH_SHORT).show();
                Log.e("UpdateItem", "Invalid input format", e);
            }
        } else {
            Toast.makeText(context, "请填写所有字段", Toast.LENGTH_SHORT).show();
        }
    }

}
```

- 创建`Student.java`文件

```java
public class Student {
    private int id;
    private String name;
    private String gender;
    private int age;
    private String birthdate;
    private String address;
    private String phone;

    public Student(int id, String name, String gender, int age, String birthdate, String address, String phone) {
        this.id = id;
        this.name = name;
        this.gender = gender;
        this.age = age;
        this.birthdate = birthdate;
        this.address = address;
        this.phone = phone;
    }

    // Getter and Setter methods
    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getGender() {
        return gender;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getBirthdate() {
        return birthdate;
    }

    public void setBirthdate(String birthdate) {
        this.birthdate = birthdate;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }
}
```
#### 5. 修改与删除功能

- 创建`student_dialog.xml`文件,实现学生信息修改

```xml
<LinearLayout  xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="@android:color/white"
    android:elevation="4dp"
    android:layout_margin="8dp">

    <EditText
        android:id="@+id/student_old_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="旧学号："
        android:inputType="number"
        android:textSize="16sp"
        android:textColor="@android:color/black"/>
    <EditText
        android:id="@+id/student_new_id"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="新学号："
        android:inputType="number"
        android:textSize="16sp"
        android:textColor="@android:color/black"/>
    <EditText
        android:id="@+id/student_name"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="姓名："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_gender"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="性别："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_age"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="年龄："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_birthdate"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="出生日期："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_address"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="家庭住址："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>

    <EditText
        android:id="@+id/student_phone"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:hint="电话号码："
        android:textSize="16sp"
        android:textColor="@android:color/black"/>
    <Button
        android:id="@+id/buttonConfirm"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="确认" />

    <Button
        android:id="@+id/buttonCancel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="取消" />
</LinearLayout>
```

- 创建`student_dialog_title.xml`文件,设置对话框标题

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
        android:text="学生信息"
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
#### 6. 学号或姓名搜索功能的实现

- 调整`MainActivity.java`文件里的`onCreateOptionsMenu`方法

```java
public class MainActivity extends AppCompatActivity implements NavigationView.OnNavigationItemSelectedListener {

    private static final String IS_LOGGED_IN = "IsLoggedIn";
    private static final String PREFS_NAME = "UserPrefs";
    private SharedPreferences sharedPreferences;
    private DatabaseHelper db;
    private DrawerLayout drawerLayout;
    private GalleryFragment galleryFragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 设置首页标题
        setTitle("pp");

        Toolbar toolbar = findViewById(R.id.toolbar);
        setSupportActionBar(toolbar);

        drawerLayout = findViewById(R.id.drawer_layout);
        NavigationView navigationView = findViewById(R.id.nav_view);
        navigationView.setNavigationItemSelectedListener(this);

        ActionBarDrawerToggle toggle = new ActionBarDrawerToggle(
                this, drawerLayout, toolbar, R.string.navigation_drawer_open, R.string.navigation_drawer_close);
        drawerLayout.addDrawerListener(toggle);
        toggle.syncState();

        // Load the default fragment
        if (savedInstanceState == null) {
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.content_frame, new HomeFragment())
                    .commit();
            navigationView.setCheckedItem(R.id.nav_home);  // 设置默认选择的菜单项
        }

        db = new DatabaseHelper(this);
        sharedPreferences = getSharedPreferences(PREFS_NAME, MODE_PRIVATE);

        if (!isUserLoggedIn()) {
            showRegisterDialog();
        }
    }

    @Override
    public void onBackPressed() {
        if (drawerLayout.isDrawerOpen(GravityCompat.START)) {
            drawerLayout.closeDrawer(GravityCompat.START);
        } else {
            super.onBackPressed();
        }
    }

    // 在需要加载 GalleryFragment 时调用这个方法
    private void loadGalleryFragment() {
        galleryFragment = new GalleryFragment();
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.content_frame, galleryFragment)
                .addToBackStack(null)  // 添加到返回堆栈以便用户可以返回到之前的Fragment
                .commit();
    }
    @Override
    public boolean onNavigationItemSelected(@NonNull MenuItem item) {
        Fragment selectedFragment = null;
        int id = item.getItemId();

        if (id == R.id.nav_home) {
            selectedFragment = new HomeFragment();
        } else if (id == R.id.nav_gallery) {
            if (!isUserLoggedIn()) {
                showRegisterDialog();
                return false; // Prevent navigation
            } else {
                loadGalleryFragment();
            }
        }else if (id == R.id.nav_slideshow) {
            selectedFragment = new SlideshowFragment();
        }

        if (id == R.id.nav_personal) {
            Intent intent = new Intent(this, PersonalActivity.class);
            startActivity(intent);
        }

        if (selectedFragment != null) {
            getSupportFragmentManager().beginTransaction()
                    .replace(R.id.content_frame, selectedFragment)
                    .commit();
        }

        DrawerLayout drawer = findViewById(R.id.drawer_layout);
        drawer.closeDrawer(GravityCompat.START);
        return true;
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
                    Toast.makeText(MainActivity.this, "登录成功", Toast.LENGTH_SHORT).show();
                    dialog.dismiss();
                } else {
                    Toast.makeText(MainActivity.this, "登录失败", Toast.LENGTH_SHORT).show();
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
                            Toast.makeText(MainActivity.this, "用户名已存在", Toast.LENGTH_SHORT).show();
                        } else {
                            // 注册用户
                            db.addUser(username, password);
                            sharedPreferences.edit().putBoolean(IS_LOGGED_IN, true).apply();
                            Toast.makeText(MainActivity.this, "注册成功", Toast.LENGTH_SHORT).show();
                            dialog.dismiss();
                        }
                    } else {
                        Toast.makeText(MainActivity.this, "用户名或密码不能为空", Toast.LENGTH_SHORT).show();
                    }
                } else {
                    Toast.makeText(MainActivity.this, "两次密码输入不一致", Toast.LENGTH_SHORT).show();
                }
            }
        });
        dialog.show();
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.drawer_menu, menu);

        MenuItem searchItem = menu.findItem(R.id.action_search);
        SearchView searchView = (SearchView) searchItem.getActionView();

        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String query) {
                // 执行最终搜索
                galleryFragment.filterStudents(query);
                return true;
            }

            @Override
            public boolean onQueryTextChange(String newText) {
                // 响应搜索查询变化
                galleryFragment.filterStudents(newText);
                return true;
            }
        });

        return true;
    }
}
```