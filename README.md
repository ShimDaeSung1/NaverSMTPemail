# SMTP를 활용한 이메일 전송하기

* SMTP를 활용한 이메일 전송하기(네이버 SMTP)

* 프로젝트 구상
  * 이메일을 전송하려면 SMTP서버가 필요한데, 포털 사이트인 네이버 메일 서버를 이용한다.
  
* 네이버 SMTP설정
  - 네이버 본인계정 로그인
  - [메일] 메뉴 접속
  - 환경설정 클릭
  - POP3/IMAP 설정 클릭

![image](https://user-images.githubusercontent.com/86938974/166616723-2e638ab8-765c-4a13-905a-3cf20d8c280b.png)


  - [POP3/STMP 설정] 탭에서 다음과 같이 선택

![image](https://user-images.githubusercontent.com/86938974/166616913-a663975f-f76c-49f4-a239-cecba9af3086.png)

* 이메일 전송 프로그램 작성
  * 이메일 작성 페이지 

```
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>SMTP 이메일 전송</title>
</head>
<body>
<h2>이메일 전송하기</h2>
<form method = "post" action = "SendProcess.jsp">
<table border = "1">
	<tr>
		<td>보내는 사람 : <input type = "text" name="from" value=""/></td> 
	</tr>
	<tr>
		<td>받는 사람 : <input type = "text" name = "to" value=""/>
	</tr>
	<tr>
		<td>제목 : <input type="text" name="subject" size="50" value=""/>
	</tr>
	<tr>
		<td>
			형식 :
			<input type = "radio" name="format" value="text" checked/> Text
			<input type = "radio" name="format" value="html"/>HTML
		</td>
	</tr>
	<tr>
		<td>
			<textarea name="content" cols="60" rows="10"></textarea>
		</td>
	</tr>
	<tr>
		<td>
		<button type="submit">전송하기</button>
		</td>
	</tr>
</table>
</form>
</body>
</html>
```
- 이메일 보낼 때의 형식은 순수 텍스트나 HTML형식 선택 가능



* 이메일 전송 클래스 작성
  * 라이브러리 설치
  * 자바메일(javamail): 메일 서비스와 관련한 전반적인 기능 수행
  * 자바빈즈 액티베이션 프레임워크 : 자바메일 API가 MIME 데이터를 관리하기 위해 사용
  
- https://mvnrepository.com/artifact/javax.mail/mail 접속
![image](https://user-images.githubusercontent.com/86938974/166617518-f6f4fb65-55d0-47e5-bda9-a964a540f331.png)
- 1.4.7 사용
![image](https://user-images.githubusercontent.com/86938974/166617551-54bf147d-0368-4d64-b336-070b6527a04c.png)
- jar파일 다운

- https://mvnrepository.com/artifact/javax.activation/activation/1.1.1 접속
![image](https://user-images.githubusercontent.com/86938974/166617644-f5dcd7ae-214a-4e8b-b770-7fb37e3bed31.png)
- jar 파일 다운

- jar파일을 WEB-INF 하위의 lib에 복사

![image](https://user-images.githubusercontent.com/86938974/166623912-e52b3cac-e6bd-4c26-8469-30efce23f1f4.png)

- 자바에서 메일 전송은 다음 순서로 이루어진다.
  - 세션 생성(메일 서버에 사용자 인증)
  - 메시지 작성
  - 전송

이메일 전송 클래스 작성
![image](https://user-images.githubusercontent.com/86938974/166624489-f5e10485-1175-4fd0-bb75-fcd8c5f984bd.png)


```
package smtp;

import java.util.Map;
import java.util.Properties;

import javax.mail.Authenticator;
import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;

// 네이버 SMTP 서버를 통해 이메일을 전송하는 클래스
public class NaverSMTP {
    private final Properties serverInfo; // 서버 정보
    private final Authenticator auth;    // 인증 정보

    public NaverSMTP() {
        // 네이버 SMTP 서버 접속 정보
        serverInfo = new Properties();
        serverInfo.put("mail.smtp.host", "smtp.naver.com");
        serverInfo.put("mail.smtp.port", "465");
        serverInfo.put("mail.smtp.starttls.enable", "true");
        serverInfo.put("mail.smtp.auth", "true");
        serverInfo.put("mail.smtp.debug", "true");
        serverInfo.put("mail.smtp.socketFactory.port", "465");
        serverInfo.put("mail.smtp.socketFactory.class",
                "javax.net.ssl.SSLSocketFactory");
        serverInfo.put("mail.smtp.socketFactory.fallback", "false");

        // 사용자 인증 정보
        auth = new Authenticator() {
            @Override
            protected PasswordAuthentication getPasswordAuthentication() {
                return new PasswordAuthentication("네이버 아이디", "네이버 패스워드");
            }
        };
    }

    // 주어진 메일 내용을 네이버 SMTP 서버를 통해 전송합니다.
    public void emailSending(Map<String, String> mailInfo) throws MessagingException {
        // 1. 세션 생성
        Session session = Session.getInstance(serverInfo, auth);
        session.setDebug(true);

        // 2. 메시지 작성
        MimeMessage msg = new MimeMessage(session);
        msg.setFrom(new InternetAddress(mailInfo.get("from")));     // 보내는 사람
        msg.addRecipient(Message.RecipientType.TO,
                         new InternetAddress(mailInfo.get("to")));  // 받는 사람
        msg.setSubject(mailInfo.get("subject"));                    // 제목
        msg.setContent(mailInfo.get("content"), mailInfo.get("format"));  // 내용

        // 3. 전송
        Transport.send(msg);
    }
}

```

- 네이버 SMTP 정보와 인증 정보는 변하지 않기 때문에 final 인스턴스 변수로 만든다. 이 둘의 값은 생성자에서 할당한다.
- emailSending()은 '세션 생성' -> '메시지 작성' -> '전송 순서'로 이메일을 전송해주는 메서드다.

* HTML 형식 메일 템플릿 준비
  * 메일 본문을 HTML형식으로 전송하려면 기본적인 HTML 문서의 형태를 갖춘다.

![image](https://user-images.githubusercontent.com/86938974/166625766-fb895e96-9a78-419e-b034-a9d16088c737.png)


```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<h2>메일 템플릿</h2>
	<table border=1 width="100%">
		<tr>
			<td width = "50">내용</td>
			<td>__CONTENT__</td> <!-- 실제 메일 내용으로 대체 -->
		</tr>
		<tr>
			<td> 이미지</td>
			<td><img src="https://github.com/goldenrabbit2020/musthave_jsp/blob/main/GOLDEN-RABBIT_LOGO_150.png?raw=true"
			alt="골든래빗"/></td>
		</tr>
	</table>
</body>
</html>
```
* 이메일 전송 처리 페이지 작성
  * EmailSendMain.jsp에서 전송하기 버튼을 누르면 이 JSP로 전달되도록 설정

![image](https://user-images.githubusercontent.com/86938974/166625827-83028494-d2a5-4a22-86d3-8e03e93a9428.png)

```
<%@ page import ="java.io.BufferedReader" %>
<%@ page import ="java.io.FileReader" %>
<%@ page import ="java.util.HashMap" %>
<%@ page import ="java.util.Map" %>
<%@ page import ="smtp.NaverSMTP" %>
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%
//폼값(이메일 내용)저장
Map<String, String> emailInfo = new HashMap<String, String>();
emailInfo.put("from", request.getParameter("from")); //보내는 사람
emailInfo.put("to", request.getParameter("to")); //받는 사람
emailInfo.put("subject", request.getParameter("subject"));

//내용은 메일 포맷에 따라 다르게 처리
String content = request.getParameter("content"); //내용
String format = request.getParameter("format"); //메일 포맷(text or html)
if(format.equals("text")){
	//텍스트 포맷일 때는 그대로 저장
	emailInfo.put("content", content);
	emailInfo.put("format", "text/plain;charset=UTF-8");
}
else if(format.equals("html")){
	//HTML 포맷일 때는 HTML 형태로 변환해 저장
	content = content.replace("\r\n", "<br/>"); // 줄바꿈을 HTML 형태로 수정
	String htmlContent = ""; //HTML용으로 변환된 내용을 담을 변수
	try {
		// HTML메일용 템플릿 파일 읽기
		String templatePath = application.getRealPath("/16EmailSend/MailForm.html");
		BufferedReader br  = new BufferedReader(new FileReader(templatePath));
		
		//한 줄씩 읽어 htmlContent 변수에 저장
		String oneLine;
		while((oneLine = br.readLine())!= null){
			htmlContent += oneLine + "\n";
		}
		br.close();
	}
	catch(Exception e){
		e.printStackTrace();
	}
	
	//템플릿의 "__CONTENT__" 부분을 대체
	htmlContent = htmlContent.replace("__CONTENT__", content);
	//변환된 내용을 저장
	emailInfo.put("content",htmlContent);
	emailInfo.put("format", "text/html;charset=UTF-8");
}

try{
	NaverSMTP smtpServer = new NaverSMTP(); // 메일 전송 클래스 생성
	smtpServer.emailSending(emailInfo);
	out.print("이메일 전송 성공");
}
catch(Exception e){
	out.print("이메일 전송 실패");
	e.printStackTrace();
}
%>

<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>

</body>
</html>
```
- 전달받은 제목과 내용 등의 폼값을 Map 타입인 emailContent 변수에 저장
- HTML메일용 템플릿인 MailForm.html 파일을 한 줄씩 읽어서 htmlContent에 저장한다.
- 이 템플릿에서 __CONTENT__ 부분을 메일 내용으로 변경해 emailContent에 저장한다. 전송요청

* 동작 확인
  * 텍스트 형식
![image](https://user-images.githubusercontent.com/86938974/166626954-96311a9f-ee1d-43f3-87b1-2dc3bc39d643.png)
![image](https://user-images.githubusercontent.com/86938974/166627137-210ede97-6b03-41a6-af51-783e0bd3a25c.png)

  * HTML 형식
![image](https://user-images.githubusercontent.com/86938974/166627232-45328f77-3547-47e0-abde-cb1a74f359d8.png)
![image](https://user-images.githubusercontent.com/86938974/166627288-206364d8-35d1-4fb2-8ca4-374d57c5e619.png)




  
  
