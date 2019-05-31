---
layout: post
title:  "간단한 junit 테스트"
date:   2019-06-01 07:39:42 +0900
categories: junit java
background: '/img/posts/vue-express.png'
comments : true
---
간단한 Rest 서버를 만들었으니  
이제 또 간단하게 junit으로 테스트 해보자.

``` java
package com.example.demo;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertTrue;

import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.HttpURLConnection;
import java.net.URL;

import org.json.simple.JSONArray;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.junit.Test;

public class junitTest {
	
	@Test
	public void validateData() throws Exception {
		String url = "http://localhost:8080/subject";
		
		URL obj = new URL(url);
		HttpURLConnection con = (HttpURLConnection) obj.openConnection();

		con.setRequestMethod("GET");

		con.setRequestProperty("User-Agent", "Mozilla/5.0");

		int responseCode = con.getResponseCode();
		System.out.println("\n요청  URL: " + url);
		System.out.println("Response Code : " + responseCode);
	
		BufferedReader in = new BufferedReader(
		        new InputStreamReader(con.getInputStream()));
		String inputLine;
		StringBuffer response = new StringBuffer(); 

		while ((inputLine = in.readLine()) != null) {
			response.append(inputLine);
		}
		in.close();
		
		JSONArray jsonArr = (JSONArray)new JSONParser().parse(response.toString());
		
		System.out.println(jsonArr);
		
		JSONObject resultData = (JSONObject)jsonArr.get(0);
		
		System.out.println(resultData.get("student_name")); 
		
		assertEquals("양찬", resultData.get("student_name"));
		
//		for (int i = 0; i < arrayObj.size(); i++) {
//			Object beforeObj = arrayObj.get(i);
//			jsonObj = (JSONObject) beforeObj;
//			System.out.println(jsonObj.get("student_name"));
//		}

		
	}

}


```




