# AWS_Study
# 스스로 구축하는 AWS 클라우드 인프라 기본편

****







## 1. 서버리스 웹 호스팅과 Cloud Front로 웹 가속화 구성하기



### 1.1 아키텍처에 구현할 기술

서버가 없어도 구성이 가능한 정적 웹 호스팅을 만들고, 웹 속도를 높이기 위하여 콘텐츠
전송 네트워크(CDN) 서비스를 연동합니다. 



![CloudFront](CloudFront.png)



​	**- 필요 AWS 서비스**

​		Amazon S3
​		Amazon CloudFront (AWS Edge Location은 전세계 160군데에 있음)



- **기타 필요사항**
  실습을 위한 간단한 HTML파일 또는 소스



- **아키텍쳐 구성**
  인터넷 > 클라우드프론트(CDN) > S3 Bucket(오브젝트 스토리지,웹상에서 콘텐츠 바로 제공가능)

****



## 아키텍쳐 구현 순서

1. S3 정적 웹 호스팅 구성하기
  1.1  S3 Bucket 생성(디폴트 옵션대로 설치,서버없이 웹 호스팅)

  1.1.1 S3버킷 생성후 Permission에 버킷 Policy를 생성/편집해서 json 형태로 기입

  1.1.2 Bucket Policy editor에서 Resource 부분에서 경로밑에 /*를 기입해야 S3 버킷의 모든 콘텐츠에 접근가능해짐

  1.1.3 속성과 Object URL이 업로드한 콘텐츠에 명시되있음

  1.2. 정적 웹 사이트 호스팅 활성화
  1.3. 웹 사이트 엔드포인트 테스트



2. CloudFront를 이용해 웹 사이트 속도 높이기
   2.1. CloudFront 배포만들기

   2.1.1 Select delivery method : WEB , RTMP 2가지 중 WEB 선택
   2.1.2 Create distribution 선택
   2.1.2.1 Origin Domain Name 선택(1번에서 생성한 S3버킷 선택)

   2.1.2.2 Origin ID : s3 bucket id ,Origin Connection Attempts :3 , Origin Connection Timeout : 10  

   2.1.2.3 Default Cache Behavior Settings {

   ```
   Path Pattern : Default (*)
   Viewer Protocol Policy : HTTP AND HTTPS
   Allowed HTTP Methods : GET,HEAD 
   Cached HTTP Methods GET,HEAD (Cached by default)
   Cache and Origin request Settings : Use a cache policy and origin request policy 
   Cache Policy : Managed-CachingOptimized 
   Smooth Streaming : No(default)
   Restrict Viewer Access (Use Signed URLS or Signed Cookies)
   Compress Objects Automatically : No
   ```

   }

   

   2.1.2.4 Distribution Settings {

   ```
   Price Class : 지역선택을 하면됨
   ( 
   1. Use Only U.S , Canada and Europe  (미국,캐나다 유럽만 사용 )
   2. Use U.S , Canada, Europe, Asia, Middle East(중동) and Africa (미국,캐나다,유럽,아시아, 중동,아프리카)
   3. Use All Edge Locations (Best Perfomance)
   )
   AWS WAF Web ACL : NONE 
   Alternate Domain Names (CNAMES) : ' '
   SSL Certificate : Default CloudFront Certificate (*.cloudfront.net)
   Supported HTTP Versions : HTTP/2. 
   Default Root Object : ' '
   Logging : ' '
   Enable IPv6 : 체크
   comment : ' '
   Distribution State : Enabled
   
   ```

   }

   

   2.2. 생성된 CloudFront 도메인 확인

   - 보통생성 시간은 약 5분~10분정도 소요되며,  위의 설명을 따라서, 옵션대로 셋팅하면 전세계 EDGE Location에 생성되는 것이다. 





****

- **용어 정리**
  CDN: Contents Delivery Networks의 줄임말로 정적콘텐츠를 캐싱해서 보여주는 서비스

​		Object storage : 하나의 파일과 그 파일을 설명하는 메타데이터까지 오브젝트라고 함
​		(S3는 오브젝트를 버킷이라고 하는 저장공간에 저장함, 디렉토리의 개념임(PC), 버킷의 권한
​		조정을 통해 오브젝트를 업로드하고, 삭제하는것을 조정함
