---
layout: post
title:  "초보자를 위한 간단 Rest API 및 디비 연동"
date:   2019-06-01 07:39:42 +0900
categories: rest server java
background: '/img/posts/vue-express.png'
comments : true
---
초보자들에게 처음부터 클래스 분할을 통한 방법은 헷갈릴 수 있으니  
우선 하나의 파일에 통합해서 해보고 이후에 클래스 분할하도록 해보자.  

``` java
package com.example.demo;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.ArrayList;

import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping(value = "subject")
public class UrlController {
	
	private static final String DRIVER = "com.mysql.jdbc.Driver";
	private static final String URL = "jdbc:mysql://127.0.0.1:3306/heets?serverTimezone=UTC"; // 접속할 db명 테이블이랑 헷갈리지 말것
	private static final String USER = "root";
	private static final String PW = "1234";
	
	Connection conn = null;
	Statement state = null;	
	String sql;
	ArrayList subjectList = new ArrayList();
	
	JSONArray jsonArray = new JSONArray();

	public UrlController() {
		try {
			
			Class.forName(DRIVER);
			conn = DriverManager.getConnection(URL, USER, PW);
			System.out.println("DB 연동 완료");
			state = conn.createStatement();
			
		} catch(Exception ex) {
			ex.printStackTrace();
		}
	}
	public void getDBData(String query) {
		jsonArray.clear();
		
		try {
			ResultSet rs = state.executeQuery(query);
			
			while(rs.next()) {
				JSONObject json = new JSONObject();	
				json.put("no", rs.getInt("no"));
				json.put("student_name", rs.getString("student_name"));
				json.put("subject", rs.getString("subject"));
				json.put("score", rs.getInt("score"));
				json.put("no", rs.getInt("student_number"));
				jsonArray.add(json);
			}
			rs.close();
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}

	@GetMapping() 
	public JSONArray getTotalSubject() {
		sql = "SELECT * FROM subject"; 
		getDBData(sql);
		return jsonArray;
	}
	
	@GetMapping("/{columnName}={subname}") 
	public JSONArray getSubjectByName(@PathVariable("columnName") String column, @PathVariable("subname") String name) {
		sql = "SELECT * FROM subject where " + column + "='" + name + "'";
		getDBData(sql);
		return jsonArray;
	}
}

```

기본적인 rest API 셋팅과 디비 연동을 해보았다.  


