# File Picker
In this application, we call default file manger of android device for choosing files.


## Tech Stack

Java

## Source code

- #### Declare permission for using external storage
- #### Set requestLegacyExternalStorage falg into true for API 29 or up to use external storage

AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="ir.mjahanbazi.fileChooser">

    <application
        android:requestLegacyExternalStorage="true"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher"
        android:supportsRtl="true"
        android:theme="@style/Theme.FileChooser">
        <activity
            android:name="ir.mjahanbazi.fileChooser.MainActivity"
            android:configChanges="orientation|screenSize|screenLayout|keyboardHidden">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
</manifest>
```
#### check for write and read permission of external storage
MainActivity.java
```java
import android.Manifest;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.database.Cursor;
import android.net.Uri;
import android.os.Build;
import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.FragmentTransaction;

public class MainActivity extends AppCompatActivity {
    public static final int REQUEST_CODE_SOME_FEATURES_PERMISSIONS = 1;
    private static StringBuilder str;

    private String getPath(Uri uri) {
        if ("content".equalsIgnoreCase(uri.getScheme())) {
            String[] projection = {"_data"};
            Cursor cursor = null;
            try {
                cursor = getContentResolver().query(uri, projection, null, null, null);
                int column_index = cursor.getColumnIndexOrThrow("_data");
                if (cursor.moveToFirst()) {
                    return cursor.getString(column_index);
                }
            } catch (Exception e) {
                return "";
            }
        } else if ("file".equalsIgnoreCase(uri.getScheme())) {
            return uri.getPath();
        }
        return null;
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
        if (hasPermission()) {
            ft.replace(R.id.your_placeholder, new FileChooserFragment());
            ft.commit();
        } else {
            ft.replace(R.id.your_placeholder, new RequestPermissionFragment());
            ft.commit();

        }
    }

    private boolean hasPermission() {
        boolean has = true;
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (checkSelfPermission(Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                has = false;
            }
            if (checkSelfPermission(Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED) {
                has = false;
            }
        }
        return has;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        if (REQUEST_CODE_SOME_FEATURES_PERMISSIONS == requestCode) {
            for (int i = 0; i < permissions.length; i++) {
                int grantResult = grantResults[i];
                if (grantResult == PackageManager.PERMISSION_GRANTED) {
                    FragmentTransaction ft = getSupportFragmentManager().beginTransaction();
                    ft.replace(R.id.your_placeholder, new FileChooserFragment());
                    ft.commit();
                }
            }
        }
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
        switch (requestCode) {
            case FileChooserUtils.FILE_SELECT_CODE:
                str = new StringBuilder();
                if (resultCode == RESULT_OK) {
                    Uri uri = data.getData();
                    String path = null;
                    path = getPath(uri);
                    FileChooserUtils.filePath.setText("Path of file: " + path);
                }
                break;
        }
    }
}
```
FileChooserFragment.java
```java
import android.content.Intent;
import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.Button;

import androidx.annotation.Nullable;
import androidx.core.app.ActivityCompat;
import androidx.fragment.app.Fragment;


public class FileChooserFragment extends Fragment {
    private View.OnClickListener onClickListener = new View.OnClickListener() {
        @Override
        public void onClick(View view) {
        }
    };

    public FileChooserFragment() {
    }

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, Bundle savedInstanceState) {
        View rootView = inflater.inflate(R.layout.file_chooser, container, false);
        return rootView;
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        Button b1 = view.findViewById(R.id.b1);
        FileChooserUtils.filePath = view.findViewById(R.id.path);
        b1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent();
                intent.setType("*/*");
                intent.setAction(Intent.ACTION_GET_CONTENT);
                intent.addCategory(Intent.CATEGORY_OPENABLE);
                ActivityCompat.startActivityForResult(getActivity(), Intent.createChooser(intent, "Select a file"),
                        FileChooserUtils.FILE_SELECT_CODE, null);
            }
        });
    }

}
```
## Contact

mjahanbazi@protonmail.com