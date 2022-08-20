---
title: "[Java] File Properties"
date: 2017-07-01 21:15:03 +0700
categories: [Java]
tags: [java, file properties]
---

Hi mọi người!
<br/>
Chắc hẳn mọi người đã từng làm việc với CSDL hay networks. Có những thông số thường xuyên thay đổi như tên CSDL, cổng, tài khoản, người dùng, mật khẩu...
<br/>
Nếu những thông tin này bạn fix cứng trong code thì có vấn đề đặt ra là nếu muốn thay đổi bạn phải mở file code lên và tìm đến file chúng ta đã cấu hình để thay đổi ? Làm như vậy phức tạp và khá lằng nhằng.

# File Properties
Thay vì hardcode thì ta sẽ lưu thông tin cấu hình vào 1 file để có thể dễ dàng cho việc thay đổi các thông tin cấu hình. 
Trong java cung cấp lớp **java.util.Properties** cung cấp một số phương thức :
* *Nạp 1 cặp key/value vào đối tượng Properties từ luồng*
* *Lấy value từ key của nó*
* *Lấy ra danh sách key và value*
* *Lưu properties vào luồng*

# Cấu trúc file properties
File properties là 1 dạng file text theo cấu trúc `key=value`, cách sử dụng rất đơn giản.
Ví dụ: 
~~~file
 # Config dang nhap he thong
 username=test123
 code=16996
~~~
 **Chú ý :** 
 * key và value viết sát nhau bởi dấu =
 * Key không được chứa khoảng trắng
 * Value có thể chứa khoảng trắng

# Demo
~~~java
package FileProperties;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.util.Enumeration;
import java.util.Properties;

/**
 *
 * @author Hades
 */
public class PropertiesFile {
    
    public static void main(String[] args) {
        Properties props = new Properties();
        OutputStream outStream = null;
        InputStream inStream = null;
        
        writeToPropsFile(props,outStream);
        loadPropsFile(props,inStream);
        printEveryThing(props,inStream);
        
    }

    private static void writeToPropsFile(Properties props, OutputStream outStream) {
        try {
            outStream = new FileOutputStream("config.properties");
             // Note :Nếu đường dẫn file không được xác định, tập tin sẽ được lưu trong thư mục gốc của project
             
            props.setProperty("port","8983");
            props.setProperty("user","admin");
            props.setProperty("pass","admin");
            
            props.store(outStream,null);
        } 
        catch (Exception e) {
            System.out.println(e.getCause());
        }
        finally {
            if(outStream != null) {
                try {
                    outStream.close();
                }
                catch (Exception e) {
                    System.out.println(e.getCause());
                }
            }
        }
    }

    private static void loadPropsFile(Properties props, InputStream inStream) {
        try {
            inStream = new FileInputStream("config.properties");
            
            props.load(inStream);
            System.out.println(props.get("port"));
            System.out.println(props.get("user"));
            System.out.println(props.get("pass"));
            
        } 
        catch (Exception e) {
            System.out.println(e.getCause());
        }
        finally {
            if(inStream != null) {
                try {
                    inStream.close();
                } 
                catch (IOException ex) {
                    System.out.println(ex.getCause());
                }
            }
        }
    }

    private static void printEveryThing(Properties props, InputStream inStream) {
        try {
            inStream = new FileInputStream("config.properties");
            props.load(inStream);
            Enumeration<?> e = props.propertyNames();
            while(e.hasMoreElements()) {
                String key = (String) e.nextElement();
                String value = props.getProperty(key);
                System.out.println("KEY: " + key + " - " + "VALUE: "+value);
            }
        } 
        catch (IOException ex) {
            System.out.println(ex.getCause());
        }
    }
}
~~~
Run lần lượt từng method kết quả trả về :

`config.properties`
~~~file
# Sat Jan 07 23:22:03 ICT 2017
user=admin
port=8983
pass=admin
~~~

`output`
~~~file
8983
admin
admin
~~~

`output`
~~~file
KEY: user - VALUE: admin
KEY: port - VALUE: 8983
KEY: pass - VALUE: admin
~~~

## Conclusion 

Chúc các bạn coding vui vẻ :D
