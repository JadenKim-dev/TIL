# gradle wrapper

## gradle이란

gradle은 java 프로젝트에 사용하는 빌드 도구이다.  
프로젝트에 build.gradle 파일을 작성해서 어떤 의존성을 프로젝트에서 사용할지를 명시하면, gradle에서 필요한 의존성을 다운 받고 이를 참고하여 프로젝트 코드를 빌드하는 작업을 한다.  
최종 결과로 packaging을 통해 jar 등의 실행 가능한 파일로 만들어준다.  
(이 외에도 여러 Task를 정의해서 다양한 작업을 할 수 있다.)

프로젝트 빌드는 다음의 명령어를 사용한다.

```bash
gradle build
```

## gradle wrapper란

gradle, maven 등의 build tool을 사용할 때 해당 툴을 감싼 wrapper를 많이 사용한다.

gradle wrapper를 사용하는 이유는, 프로젝트 구현 당시 사용했던 동일한 jdk/jre 환경을 사용해서 빌드하기 위해서이다.  
일반 gradle 명령어를 사용해서 프로젝트를 빌드하면 시스템에 설치되어있는 jdk를 사용해서 빌드를 수행하게 된다.  
gradle wrapper를 사용하면 build.gradle을 참고하여 프로젝트가 작성된 환경과 동일한 jdk bin을 다운로드 받고, 이를 이용하여 빌드를 수행하는 스크립트를 gradlew 파일에 작성한다.

해당 gradlew 파일을 이용하여 빌드를 수행하는 스크립트는 아래와 같다.

```bash
gradlew build
```

위 명령어를 실행하게 되면, gradlew에서는 지정한 jdk의 binary 파일이 다운로드 되어 있는지 확인하고, 없다면 설치하게 된다.  
이 때 jdk의 binary 파일은 `~/.gradle/wrapper/dists`의 하위 폴더에 다운로드 된다.  
해당 시스템에서 다양한 버전을 사용하는 자바 프로젝트를 빌드했다면 여러 버전의 jdk가 해당 path에 설치된다.

해당 wrapper 관련 설정은 <프로젝트 루트>/gradle/wrapper/gradle-wrapper.properties에서 수행할 수 있다.  
프로젝트 루트 위치에서 `gradle wrapper` 명령어를 실행하면 wrapper에 필요한 기본 디렉토리 구조가 생성된다.
