---
title: "Android通过webservice连接SQLServer详细教程"
date: 2020-10-4T23:36:51+08:00
comments: true #是否可评论
toc: true #是否显示文章目录
categories: 
 - 技术 #分类
series: golang
tags: #标签
 - android
 - webservice
 - SQLServer
mp3: http://aod.cos.tx.xmcdn.com/group60/M09/2E/AD/wKgLeVzHz6GgU9p0ABqur_SQyPs293.mp3
keywords:
description: 数据库＋服务器＋客户端 
cover: /img/wallpaper-2415699.jpg
---

本人研究生阶段的第二个项目是**“安全标准化手机App”**，周期约5个月，主要工作是自学Android并完成了这个系统的搭建和手机App的开发。开发阶段印象最深的就是这篇文章，所以重新编辑，精简步骤并记录一些爬坑经验，希望能够帮助需要的朋友快速入门进阶。

## 1.连接DB主流的两种方式： ##
1）使用jtds直接访问DB数据库（参考：https://blog.csdn.net/feidie436/article/details/77532194/）<br>
2）以webservice方式进行远程访问数据库，使用ksoap2进行数据读取及json进行数据解析
## 2.理解以下两点： ##
1）android无法直接连接SQLServer，sqlserver安装之后有多大，android程序是跑在手机上的，想让程序直接访问sqlserver，那手机要多大的内存？android内置sqlite轻量级数据库，我们只需要一个**“桥梁”**将数据进行交换存储。

2）本文是通过一个**“桥梁”**——webservice来间接访问SQLServer的，当然还有其他方法，感兴趣的朋友可以自行google。

**教程会拿一个具体的例子来讲，一步一步来，也许细节上还可以继续加工，流程没问题。**

## 3.本教程有五个部分: ##

- **项目说明**

- **开发环境部署**
·数据库设计__<br>
·服务器端程序设计__<br>
·客户端(android端)程序设计<br>**



###（1）项目说明 ###
这个项目意在实现一个简单的android连接Sqlserver的功能。

就做一个简单的库存管理功能，包括对仓库内现有货物的查看、货物信息的增加&删除。

###（2）开发环境的部署 ###

**操作系统:Windows7-64bit 旗舰版**

需要开启IIS服务,家庭版缺失这个服务<br>

**android端:eclipse + ADT集成开发环境**

相信看到这个教程的基本都知道如何做这些了.如果真的是有哪位同学android开发环境没有配置好而来看这篇教程,请先移步->www.google.com<>
现在主流是android studio
**
服务器端:VisualStudio 2010 旗舰版**

这个是用来写website/webservice的,开发语言使用C#(即.net)

**数据库:SQLServer2008 R2**

其实这个是什么版本也无所谓吧，教程使用的都是比较基本的东西，所以版本的差异基本可以忽略。

**IIS 7.5:正确配置并开启IIS服务**

如果想将website/webservice发布出去就要开启这个服务。但是如果仅仅是在本地进行测试就不需要配置，直接在VS中运行就可以。

http://wenku.baidu.com/view/95cf9fd9ad51f01dc281f1af.html这篇文库给的还是很详细的，我当初就是照着这个配置的。

数据库设计
数据库名称：StockManage

表设计

表名称：C

表说明：
https://cdn.jsdelivr.net/gh/liusven/cdn@1.0/Android%E9%80%9A%E8%BF%87webservice%E8%BF%9E%E6%8E%A5SQLServer%E8%AF%A6%E7%BB%86%E6%95%99%E7%A8%8B/1shuoming.png


下图是设计表的时候的截图。


 

向表中输入内容:

 

 

 

## 服务器端程序设计（Webservice） ##
其实服务端可以写成webservice也可以写成website，前者只是提供一种服务，而后者是可以提供用户界面等具体的页面，后者也就是咱们平时所说的“网站”。

两者的区别：

Web Service 只提供程序和接口，不提供用户界面
Web Site 提供程序和接口，也提供用户界面（网页）
由于咱们只是需要一个中介来访问sqlserver，所以写成webservice足够了。

目标：写一个Website访问Sqlserver，获取数据并转换成xml格式，然后传递给android客户端。


1.新建一个Webservice工程


2.视图 -> 其它窗口 -> 服务器资源管理器


3.右键数据连接 -> 添加连接


4.选择Microsoft Sqlserver


5.如下图所示选择（可以点击测试连接来检测连接是否成功，然后点击确定）


6.数据库的查看和编辑也可以在VS中进行了


7.先查看一下数据库属性并记录下连接属性


8.新建一个类DBOperation，代码如下：

	using System;
	using System.Data;
	using System.Configuration;
	using System.Linq;
	using System.Web;
	using System.Web.Security;
	using System.Web.UI;
	using System.Web.UI.HtmlControls;
	using System.Web.UI.WebControls;
	using System.Web.UI.WebControls.WebParts;
	using System.Xml.Linq;
	using System.Data.SqlClient;
	using System.Text.RegularExpressions;
	using System.Collections;
	using System.Collections.Generic;
 
	namespace StockManageWebservice
	{
    /// <summary>
    /// 一个操作数据库的类，所有对SQLServer的操作都写在这个类中，使用的时候实例化一个然后直接调用就可以
    /// </summary>
    public class DBOperation:IDisposable
    {
        public static SqlConnection sqlCon;  //用于连接数据库
 
        //将下面的引号之间的内容换成上面记录下的属性中的连接字符串
        private String ConServerStr = @"Data Source=BOTTLE-PC;Initial Catalog=StockManage;Integrated Security=True";
        
        //默认构造函数
        public DBOperation()
        {
            if (sqlCon == null)
            {
                sqlCon = new SqlConnection();
                sqlCon.ConnectionString = ConServerStr;
                sqlCon.Open();
            }
        }
         
        //关闭/销毁函数，相当于Close()
        public void Dispose()
        {
            if (sqlCon != null)
            {
                sqlCon.Close();
                sqlCon = null;
            }
        }
        
        /// <summary>
        /// 获取所有货物的信息
        /// </summary>
        /// <returns>所有货物信息</returns>
        public List<string> selectAllCargoInfor()
        {
            List<string> list = new List<string>();
 
            try
            {
                string sql = "select * from C";
                SqlCommand cmd = new SqlCommand(sql,sqlCon);
                SqlDataReader reader = cmd.ExecuteReader();
 
                while (reader.Read())
                {
                    //将结果集信息添加到返回向量中
                    list.Add(reader[0].ToString());
                    list.Add(reader[1].ToString());
                    list.Add(reader[2].ToString());
 
                }
 
                reader.Close();
                cmd.Dispose();
 
            }
            catch(Exception)
            {
 
            }
            return list;
        }
 
        /// <summary>
        /// 增加一条货物信息
        /// </summary>
        /// <param name="Cname">货物名称</param>
        /// <param name="Cnum">货物数量</param>
        public bool insertCargoInfo(string Cname, int Cnum)
        {
            try
            {
                string sql = "insert into C (Cname,Cnum) values ('" + Cname + "'," + Cnum + ")";
                SqlCommand cmd = new SqlCommand(sql, sqlCon);
                cmd.ExecuteNonQuery();
                cmd.Dispose();
 
                return true;
            }
            catch (Exception)
            {
                return false;
            }
        }
 
        /// <summary>
        /// 删除一条货物信息
        /// </summary>
        /// <param name="Cno">货物编号</param>
        public bool deleteCargoInfo(string Cno)
        {
            try
            {
                string sql = "delete from C where Cno=" + Cno;
                SqlCommand cmd = new SqlCommand(sql, sqlCon);
                cmd.ExecuteNonQuery();
                cmd.Dispose();
 
                return true;
            }
            catch (Exception)
            {
                return false;
            }
        }
    }
}

9.      修改Service1.asmx.cs代码如下：
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Services;
 
namespace StockManageWebservice
{
    /// <summary>
    /// Service1 的摘要说明
    /// </summary>
    [WebService(Namespace = "http://tempuri.org/")]
    [WebServiceBinding(ConformsTo = WsiProfiles.BasicProfile1_1)]
    [System.ComponentModel.ToolboxItem(false)]
    // 若要允许使用 ASP.NET AJAX 从脚本中调用此 Web 服务，请取消对下行的注释。
    // [System.Web.Script.Services.ScriptService]
    public class Service1 : System.Web.Services.WebService
    {
        DBOperation dbOperation = new DBOperation();
 
        [WebMethod]
        public string HelloWorld()
        {
            return "Hello World";
        }
 
        [WebMethod(Description = "获取所有货物的信息")]
        public string[] selectAllCargoInfor()
        {
            return dbOperation.selectAllCargoInfor().ToArray();
        }
 
        [WebMethod(Description = "增加一条货物信息")]
        public bool insertCargoInfo(string Cname, int Cnum)
        {
            return dbOperation.insertCargoInfo(Cname, Cnum);
        }
 
        [WebMethod(Description = "删除一条货物信息")]
        public bool deleteCargoInfo(string Cno)
        {
            return dbOperation.deleteCargoInfo(Cno);
        }
    }
}


10.      运行程序(F5),会自动打开一个浏览器,可以看到如下画面:


11.  选择相应的功能并传递参数可以实现调试从浏览器中调试程序:

下图选择的是增加一条货物信息



12.  程序执行的结果:

13.另，记住这里的端口名，后面android的程序中添入的端口号就是这个：



## 客户端(android端)程序设计 ##
程序代码:

1.MainActivity

package com.bottle.stockmanage;
 
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
 
import android.app.Activity;
import android.app.Dialog;
import android.os.Bundle;
import android.view.Gravity;
import android.view.View;
import android.view.View.OnClickListener;
import android.view.Window;
import android.view.WindowManager;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ListView;
import android.widget.SimpleAdapter;
import android.widget.Toast;
 
public class MainActivity extends Activity{
 
	private Button btn1;
	private Button btn2;
	private Button btn3;
	private ListView listView;
	private SimpleAdapter adapter;
	private DBUtil dbUtil;
 
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
 
		btn1 = (Button) findViewById(R.id.btn_all);
		btn2 = (Button) findViewById(R.id.btn_add);
		btn3 = (Button) findViewById(R.id.btn_delete);
		listView = (ListView) findViewById(R.id.listView);
		dbUtil = new DBUtil();
		
		btn1.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				hideButton(true);
				setListView();
			}
		});
 
		btn2.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				hideButton(true);
				setAddDialog();
			}
		});
 
		btn3.setOnClickListener(new OnClickListener() {
			
			@Override
			public void onClick(View v) {
				hideButton(true);
				setDeleteDialog();
			}
		});
	}
 
	/**
	 * 设置弹出删除对话框
	 */
	private void setDeleteDialog() {
		
		final Dialog dialog = new Dialog(MainActivity.this);
		dialog.setContentView(R.layout.dialog_delete);
		dialog.setTitle("输入想要删除的货物的编号");
		Window dialogWindow = dialog.getWindow();
		WindowManager.LayoutParams lp = dialogWindow.getAttributes();
		dialogWindow.setGravity(Gravity.CENTER);
		dialogWindow.setAttributes(lp);
 
		final EditText cNoEditText = (EditText) dialog.findViewById(R.id.editText1);
		Button btnConfirm = (Button) dialog.findViewById(R.id.button1);
		Button btnCancel = (Button) dialog.findViewById(R.id.button2);
 
		btnConfirm.setOnClickListener(new OnClickListener() {
 
			@Override
			public void onClick(View v) {
				dbUtil.deleteCargoInfo(cNoEditText.getText().toString());
				dialog.dismiss();
				hideButton(false);
				Toast.makeText(MainActivity.this, "成功删除数据", Toast.LENGTH_SHORT).show();
			}
		});
 
		btnCancel.setOnClickListener(new OnClickListener() {
 
			@Override
			public void onClick(View v) {
				dialog.dismiss();
				hideButton(false);
			}
		});
		
		dialog.show();
	}
 
	/**
	 * 设置弹出添加对话框
	 */
	private void setAddDialog() {
 
		final Dialog dialog = new Dialog(MainActivity.this);
		dialog.setContentView(R.layout.dialog_add);
		dialog.setTitle("输入添加的货物的信息");
		Window dialogWindow = dialog.getWindow();
		WindowManager.LayoutParams lp = dialogWindow.getAttributes();
		dialogWindow.setGravity(Gravity.CENTER);
		dialogWindow.setAttributes(lp);
 
		final EditText cNameEditText = (EditText) dialog.findViewById(R.id.editText1);
		final EditText cNumEditText = (EditText) dialog.findViewById(R.id.editText2);
		Button btnConfirm = (Button) dialog.findViewById(R.id.button1);
		Button btnCancel = (Button) dialog.findViewById(R.id.button2);
 
		btnConfirm.setOnClickListener(new OnClickListener() {
 
			@Override
			public void onClick(View v) {
				
				dbUtil.insertCargoInfo(cNameEditText.getText().toString(), cNumEditText.getText().toString());
				dialog.dismiss();
				hideButton(false);
				Toast.makeText(MainActivity.this, "成功添加数据", Toast.LENGTH_SHORT).show();
			}
		});
 
		btnCancel.setOnClickListener(new OnClickListener() {
 
			@Override
			public void onClick(View v) {
				dialog.dismiss();
				hideButton(false);
			}
		});
		dialog.show();
	}
 
	/**
	 * 设置listView
	 */
	private void setListView() {
 
		listView.setVisibility(View.VISIBLE);
 
		List<HashMap<String, String>> list = new ArrayList<HashMap<String, String>>();
 
		list = dbUtil.getAllInfo();
 
		adapter = new SimpleAdapter(
				MainActivity.this, 
				list, 
				R.layout.adapter_item, 
				new String[] { "Cno", "Cname", "Cnum" }, 
				new int[] { R.id.txt_Cno, R.id.txt_Cname, R.id.txt_Cnum });
 
		listView.setAdapter(adapter);
 
	}
 
	/**
	 * 设置button的可见性
	 */
	private void hideButton(boolean result) {
		if (result) {
			btn1.setVisibility(View.GONE);
			btn2.setVisibility(View.GONE);
			btn3.setVisibility(View.GONE);
		} else {
			btn1.setVisibility(View.VISIBLE);
			btn2.setVisibility(View.VISIBLE);
			btn3.setVisibility(View.VISIBLE);
		}
 
	}
 
	/**
	 * 返回按钮的重写
	 */
	@Override
	public void onBackPressed()
	{
		if (listView.getVisibility() == View.VISIBLE) {
			listView.setVisibility(View.GONE);
			hideButton(false);
		}else {
			MainActivity.this.finish();
		}
	}
}

2.HttpConnSoap

(改类已经过时，更多请参照

http://blog.csdn.net/zhyl8157121/article/details/8709048)

package com.bottle.stockmanage;
 
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.ArrayList;
 
public class HttpConnSoap {
	public ArrayList<String> GetWebServre(String methodName, ArrayList<String> Parameters, ArrayList<String> ParValues) {
		ArrayList<String> Values = new ArrayList<String>();
		
		//ServerUrl是指webservice的url
		//10.0.2.2是让android模拟器访问本地（PC）服务器，不能写成127.0.0.1
		//11125是指端口号，即挂载到IIS上的时候开启的端口
		//Service1.asmx是指提供服务的页面
		String ServerUrl = "http://10.0.2.2:11125/Service1.asmx";
		
		//String soapAction="http://tempuri.org/LongUserId1";
		String soapAction = "http://tempuri.org/" + methodName;
		//String data = "";
		String soap = "<?xml version=\"1.0\" encoding=\"utf-8\"?>"
				+ "<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\" xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">"
				+ "<soap:Body />";
		String tps, vps, ts;
		String mreakString = "";
 
		mreakString = "<" + methodName + " xmlns=\"http://tempuri.org/\">";
		for (int i = 0; i < Parameters.size(); i++) {
			tps = Parameters.get(i).toString();
			//设置该方法的参数为.net webService中的参数名称
			vps = ParValues.get(i).toString();
			ts = "<" + tps + ">" + vps + "</" + tps + ">";
			mreakString = mreakString + ts;
		}
		mreakString = mreakString + "</" + methodName + ">";
		/*
		+"<HelloWorld xmlns=\"http://tempuri.org/\">"
		+"<x>string11661</x>"
		+"<SF1>string111</SF1>"
		+ "</HelloWorld>"
		*/
		String soap2 = "</soap:Envelope>";
		String requestData = soap + mreakString + soap2;
		//System.out.println(requestData);
 
		try {
			URL url = new URL(ServerUrl);
			HttpURLConnection con = (HttpURLConnection) url.openConnection();
			byte[] bytes = requestData.getBytes("utf-8");
			con.setDoInput(true);
			con.setDoOutput(true);
			con.setUseCaches(false);
			con.setConnectTimeout(6000);// 设置超时时间
			con.setRequestMethod("POST");
			con.setRequestProperty("Content-Type", "text/xml;charset=utf-8");
			con.setRequestProperty("SOAPAction", soapAction);
			con.setRequestProperty("Content-Length", "" + bytes.length);
			OutputStream outStream = con.getOutputStream();
			outStream.write(bytes);
			outStream.flush();
			outStream.close();
			InputStream inStream = con.getInputStream();
 
			//data=parser(inStream);
			//System.out.print("11");
			Values = inputStreamtovaluelist(inStream, methodName);
			//System.out.println(Values.size());
			return Values;
 
		} catch (Exception e) {
			System.out.print("2221");
			return null;
		}
	}
 
	public ArrayList<String> inputStreamtovaluelist(InputStream in, String MonthsName) throws IOException {
		StringBuffer out = new StringBuffer();
		String s1 = "";
		byte[] b = new byte[4096];
		ArrayList<String> Values = new ArrayList<String>();
		Values.clear();
 
		for (int n; (n = in.read(b)) != -1;) {
			s1 = new String(b, 0, n);
			out.append(s1);
		}
 
		System.out.println(out);
		String[] s13 = s1.split("><");
		String ifString = MonthsName + "Result";
		String TS = "";
		String vs = "";
 
		Boolean getValueBoolean = false;
		for (int i = 0; i < s13.length; i++) {
			TS = s13[i];
			System.out.println(TS);
			int j, k, l;
			j = TS.indexOf(ifString);
			k = TS.lastIndexOf(ifString);
 
			if (j >= 0) {
				System.out.println(j);
				if (getValueBoolean == false) {
					getValueBoolean = true;
				} else {
 
				}
 
				if ((j >= 0) && (k > j)) {
					System.out.println("FFF" + TS.lastIndexOf("/" + ifString));
					//System.out.println(TS);
					l = ifString.length() + 1;
					vs = TS.substring(j + l, k - 2);
					//System.out.println("fff"+vs);
					Values.add(vs);
					System.out.println("退出" + vs);
					getValueBoolean = false;
					return Values;
				}
 
			}
			if (TS.lastIndexOf("/" + ifString) >= 0) {
				getValueBoolean = false;
				return Values;
			}
			if ((getValueBoolean) && (TS.lastIndexOf("/" + ifString) < 0) && (j < 0)) {
				k = TS.length();
				//System.out.println(TS);
				vs = TS.substring(7, k - 8);
				//System.out.println("f"+vs);
				Values.add(vs);
			}
 
		}
 
		return Values;
	}
 
}

3.DBUtil
package com.bottle.stockmanage;
 
import java.sql.Connection;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
 
public class DBUtil {
	private ArrayList<String> arrayList = new ArrayList<String>();
	private ArrayList<String> brrayList = new ArrayList<String>();
	private ArrayList<String> crrayList = new ArrayList<String>();
	private HttpConnSoap Soap = new HttpConnSoap();
 
	public static Connection getConnection() {
		Connection con = null;
		try {
			//Class.forName("org.gjt.mm.mysql.Driver");
			//con=DriverManager.getConnection("jdbc:mysql://192.168.0.106:3306/test?useUnicode=true&characterEncoding=UTF-8","root","initial");  		    
		} catch (Exception e) {
			//e.printStackTrace();
		}
		return con;
	}
 
	/**
	 * 获取所有货物的信息
	 * 
	 * @return
	 */
	public List<HashMap<String, String>> getAllInfo() {
		List<HashMap<String, String>> list = new ArrayList<HashMap<String, String>>();
 
		arrayList.clear();
		brrayList.clear();
		crrayList.clear();
 
		crrayList = Soap.GetWebServre("selectAllCargoInfor", arrayList, brrayList);
 
		HashMap<String, String> tempHash = new HashMap<String, String>();
		tempHash.put("Cno", "Cno");
		tempHash.put("Cname", "Cname");
		tempHash.put("Cnum", "Cnum");
		list.add(tempHash);
		
		for (int j = 0; j < crrayList.size(); j += 3) {
			HashMap<String, String> hashMap = new HashMap<String, String>();
			hashMap.put("Cno", crrayList.get(j));
			hashMap.put("Cname", crrayList.get(j + 1));
			hashMap.put("Cnum", crrayList.get(j + 2));
			list.add(hashMap);
		}
 
		return list;
	}
 
	/**
	 * 增加一条货物信息
	 * 
	 * @return
	 */
	public void insertCargoInfo(String Cname, String Cnum) {
 
		arrayList.clear();
		brrayList.clear();
		
		arrayList.add("Cname");
		arrayList.add("Cnum");
		brrayList.add(Cname);
		brrayList.add(Cnum);
		
		Soap.GetWebServre("insertCargoInfo", arrayList, brrayList);
	}
	
	/**
	 * 删除一条货物信息
	 * 
	 * @return
	 */
	public void deleteCargoInfo(String Cno) {
 
		arrayList.clear();
		brrayList.clear();
		
		arrayList.add("Cno");
		brrayList.add(Cno);
		
		Soap.GetWebServre("deleteCargoInfo", arrayList, brrayList);
	}
}

4.activity_main.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent" >
 
    <ListView
        android:id="@+id/listView"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:visibility="gone" >
    </ListView>
 
    <Button
        android:id="@+id/btn_all"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_above="@+id/btn_add"
        android:layout_alignLeft="@+id/btn_add"
        android:layout_marginBottom="10dip"
        android:text="@string/btn1" />
 
    <Button
        android:id="@+id/btn_add"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerHorizontal="true"
        android:layout_centerVertical="true"
        android:text="@string/btn2" />
 
    <Button
        android:id="@+id/btn_delete"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignLeft="@+id/btn_add"
        android:layout_below="@+id/btn_add"
        android:layout_marginTop="10dip"
        android:text="@string/btn3" />
 
	</RelativeLayout>

5.adapter_item.xml
<?xml version="1.0" encoding="utf-8"?>
<TableLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content"
    android:descendantFocusability="blocksDescendants"
    android:gravity="center" >
 
    <TableRow
        android:id="@+id/classroom_detail_item_tableRow"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:gravity="center" >
 
        <TextView
            android:id="@+id/txt_Cno"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:height="40dp"
            android:textSize="14sp" >
        </TextView>
 
        <TextView
            android:id="@+id/txt_Cname"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:height="40dp"
            android:textSize="14sp" >
        </TextView>
 
        <TextView
            android:id="@+id/txt_Cnum"
            android:layout_width="80dp"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:height="40dp"
            android:textSize="14sp" >
        </TextView>
 
    </TableRow>
 
	</TableLayout>

6.dialog_add.xml
	
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >
 
    <EditText
        android:id="@+id/editText1"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/add_hint1" >
 
        <requestFocus />
    </EditText>
 
    <EditText
        android:id="@+id/editText2"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/add_hint2"
        android:inputType="number" />
 
    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >
 
        <Button
            android:id="@+id/button1"
            android:layout_width="100dip"
            android:layout_height="wrap_content"
            android:layout_marginLeft="20dip"
            android:text="@string/confirm" />
 
        <Button
            android:id="@+id/button2"
            android:layout_width="100dip"
            android:layout_height="wrap_content"
            android:layout_marginLeft="40dip"
            android:text="@string/cancel" />
    </LinearLayout>
 
	</LinearLayout>

7.dialog_delete.xml
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >
 
    <EditText
        android:id="@+id/editText1"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:ems="10"
        android:hint="@string/delete_hint" >
 
        <requestFocus />
    </EditText>
 
    <LinearLayout
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal" >
 
        <Button
            android:id="@+id/button1"
            android:layout_width="100dip"
            android:layout_height="wrap_content"
            android:layout_marginLeft="20dip"
            android:text="@string/confirm" />
 
        <Button
            android:id="@+id/button2"
            android:layout_width="100dip"
            android:layout_height="wrap_content"
            android:layout_marginLeft="40dip"
            android:text="@string/cancel" />
    </LinearLayout>
 
	</LinearLayout>

8.strings.xml
	<resources>
 
    <string name="app_name">StockManagement</string>
    <string name="menu_settings">Settings</string>
    <string name="title_activity_main">MainActivity</string>
    <string name="btn1">查看所有货物信息</string>
    <string name="btn2">增加一条货物信息</string>
    <string name="btn3">删除一条货物信息</string>
    <string name="add_hint1">输入添加的货物的名称</string>
    <string name="add_hint2">输入货物的数量</string>
    <string name="confirm">确定</string>
    <string name="cancel">取消</string>
    <string name="delete_hint">输入删除的货物的编号</string>
 
	</resources>

9.Manifest.xml
	<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.bottle.stockmanage"
    android:versionCode="1"
    android:versionName="1.0" >
 
    <uses-sdk
        android:minSdkVersion="7"
        android:targetSdkVersion="15" />
 
    <uses-permission android:name="android.permission.INTERNET" />
 
    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@android:style/Theme.NoTitleBar" >
        <activity
            android:name=".MainActivity"
            android:label="@string/title_activity_main"
            android:screenOrientation="portrait" >
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
 
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
 
	</manifest>

运行程序的效果如下图所示：





再说一下IIS，如果只是在本地进行测试等操作，是不需要使用到IIS的，但是如果想发布出去，就要配置一下IIS。


好啦，基本就是这样了。程序不是完善的，但大概的思路就是这样，用到的技术也大概就是这几样，但是每一样拿出来都够学一阵的了。




--->2012.12.02 增加内容

附上本文demo的CSDN下载地址

http://download.csdn.net/detail/zhyl8157121/4836107

--->2013.01.08 增加内容
解释一下android端如何和webservice通信的。（如何修改实例程序）

具体的更深层的东西已经在HttpSoap中封装好了，所以大家使用的时候可以直接用这个类就可以了。（我也不懂是怎么实现的……）

android调用的方法就是如DBUtil中那样，比如内嵌图片 2

其中arrayList中和brrayList中分别存放对应的webservice中“selectAllCargoInfor”方法的参数名和参数的值。

内嵌图片 3
由于webservice中的selectAllCargoInfo方法的参数为空，所以对应的，android端调用的时候，arrayList和brrayList的值就是空的。


 所以大家在使用的时候，只需要将webservice中的方法写好，然后写好DBUtil中的调用参数即可。


--->2013.03.23 增加内容
如果获取值为空，可能是返回值是复杂类型造成的，可以参考：http://blog.csdn.net/zhyl8157121/article/details/8709048


--->2014.07.18 增加内容
没想到这么长时间了还能得到大家的青睐，很欣慰。

之前收到过一些同学发来的邮件，问题大概以下几种，这里统一答复一下吧，希望给有问题的同学一个参考。

0.Webservice无法正确执行（这个没什么说的，webservice有问题）

1.Webservice用本地的电脑可以访问，但是在手机浏览器中就无法得到正确的结果（Webservice配置问题，或者IIS配置问题）

2.Webservice用本地的电脑和手机浏览器都可以正确执行，但是程序一运行就崩溃

2.1 可能情况1 - 我的程序是用Android2.1写的，Android2.3以后有一个StrictMode的问题，如果有同学是用2.3以上版本做的，可能是这个问题，具体可以搜索一下StrictMode，然后修改一下程序即可，我在这里就不献丑了。


2.2 可能情况2 - 调试的时候是用的是模拟器，IP地址当时填写的是10.0.2.2，这个地址是PC相对于模拟器的IP地址，放到真机中就不行了，真机运行程序中要填写PC的IP地址（可以将手机和PC连接在同一个局域网中做测试，也可以将程序发布到服务器上，Android程序中填写服务器的IP地址和端口号）。


3.用模拟器调试怎么都可以，但放到真机上就不行。

3.1 可能情况1 - 模拟器的版本是2.3以下的，程序运行正常，可是真机是2.3以上的，解决方法参考2.1。


3.2 可能情况2 - 放到真机运行的时候没改程序的url，IP地址错误，解决方法参考2.2。


4.SQL语句错误、按钮监听错误等问题大家细心查查就行了。


不出什么意外的话，本文不会再更新了，楼主已经很久不搞Android了，有些东西也稍微有点生疏了。


以前有些同学给我发邮件或者私信我是每个都回复的，不管问的问题我自己会不会。工作了之后就比较忙了，而且Gmail经常上不去（你懂的），有些同学的邮件就没有回复，在此向那些我没有回复的同学说声抱歉，大家都是来交流的，我绝对不是高傲或者懒散，交个朋友也好。


大家不要问QQ号了，那个东西不太适合技术交流，还是邮箱来的实在，这能让彼此的思考时间多一些。


再次留下我的邮箱：bottle.liang@gmail.com 如果有什么问题，欢迎大家来信交流。


谢谢支持，欢迎大家批评指正。



<br>
感谢文章作者[@Bottle](http://blog.csdn.net/zhyl8157121/article/details/8169172)(https://blog.csdn.net/zhyl8157121/article/details/8709048)
Android通过webservice连接SQLServer 详细教程（数据库+服务器+客户端）
<br>
Author：「刘吃人」
BGM： Cake By The Ocean - DNCE