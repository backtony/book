## 목차
- [Intro](#1)
- [형식을 맞추는 목적](#2)
- [적절한 행 길이를 유지하라](#3)
    - [신문 기사처럼 작성하라](#4)
    - [개념은 빈 행으로 분리하라](#5)
    - [세로 밀집도](#6)
    - [수직 거리](#7)
- [가로 형식 맞추기](#8)
	- [가로 공백과 밀집도](#9)
    - [가로 정렬](#10)
	- [들여쓰기](#11)
    - [가짜 범위](#12)
- [팀 규칙](#13)
- [밥 아저씨의 형식 규칙](#14)

---


## Intro <a name = "1"></a>
코드가 깔끔하고, 일관적이며, 꼼꼼하다면 전문가가 짰다는 인상을 심어줄 수 있다.  
프로그래머라면 형식을 깔끔하게 맞춰 코드를 짜야한다.  
코드 형식을 맞추기 위한 간단한 규칙을 정하고 그 규칙을 착실히 따라야 한다.  
팀으로 일한다면 팀이 합의해 규칙을 정하고 모두가 그 규칙을 따라야 한다.  
<br>

## 형식을 맞추는 목적 <a name = "2"></a>
코드 형식은 너무도 중요하기에 무시할 수 없다.  
코드 형식은 의사소통의 일환이다.  
오늘 구현한 코드의 가독성은 앞으로 바뀔 코드의 품질에 지대한 영향을 미친다.  
오랜 시간이 지나 원래 코드의 흔적을 더 이상 찾아보기 어려울 정도로 코드가 바뀌어도 맨 처음 잡아놓은 구현 스타일과 가독성 수준은 유지보수 용이성과 확장에 계속 영향을 미친다.  
__원래 코드는 사라질지라도 개발자의 스타일과 규율은 사라지지 않는다.__  
<br>

## 적절한 행 길이를 유지하라 <a name = "3"></a>
소스 코드는 얼마나 길어야 적당할까?  
책의 저자가 JUnit, FitNesse, testNg, Tomcat 등의 프로젝트를 조사해 내린 결론은 다음과 같다.  
__500줄을 넘지 않고 대부분 200줄 정도인 파일로 커다란 시스템을 구축할 수 있다.__  

### 신문 기사처럼 작성하라 <a name = "4"></a>
독자는 위에서 아래로 기사를 읽기에 좋은 신문 기사는 최상단에 기사를 몇 마디 요약하는 표제가 나온다.  
독자는 표제를 보고 기사를 읽을지 말지 결정한다.  
소스 파일도 신문 기사와 비슷하게 작성한다.  
이름은 간단하면서 설명이 가능하게 짓는다.  
소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명하고 아래로 내려갈수록 의도를 세세하게 묘사한다.  
마지막에는 가장 저차원 함수와 세부내역이 나온다.  

### 개념은 빈 행으로 분리하라 <a name = "5"></a>
코드의 각 줄은 수식이나 절을 나타내고, 일련의 행 묶음은 완결된 생각 하나를 표현한다.  
생각 사이는 빈 행을 넣어 분리해야 마땅하다.
```java
// bad
// 빈 행을 넣지 않을 경우
package fitnesse.wikitext.widgets;
import java.util.regex.*;
public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''",
		Pattern.MULTILINE + Pattern.DOTALL);
	public BoldWidget(ParentWidget parent, String text) throws Exception {
		super(parent);
		Matcher match = pattern.matcher(text); match.find(); 
		addChildWidgets(match.group(1));}
}
```
```java
// good
// 빈 행을 넣을 경우
package fitnesse.wikitext.widgets;

import java.util.regex.*;

public class BoldWidget extends ParentWidget {
	public static final String REGEXP = "'''.+?'''";
	private static final Pattern pattern = Pattern.compile("'''(.+?)'''", 
		Pattern.MULTILINE + Pattern.DOTALL
	);
	
	public BoldWidget(ParentWidget parent, String text) throws Exception { 
		super(parent);
		Matcher match = pattern.matcher(text);
		match.find();
		addChildWidgets(match.group(1)); 
	}
}
```

### 세로 밀집도 <a name = "6"></a>
줄바꿈이 개념을 분리한다면 세로 밀집도는 연관성을 의미한다.  
즉, __서로 밀집한 코드 행은 세로로 가까이 놓여야 한다.__
```java
// bad
public class ReporterConfig {
	/**
	* The class name of the reporter listener 
	*/
	private String m_className;
	
	/**
	* The properties of the reporter listener 
	*/
	private List<Property> m_properties = new ArrayList<Property>();
	public void addProperty(Property property) { 
		m_properties.add(property);
	}
```
의미없는 주석으로 변수를 떨어뜨려 놓아서 한눈에 파악이 잘 안 된다.
```java
public class ReporterConfig {
	private String m_className;
	private List<Property> m_properties = new ArrayList<Property>();
	
	public void addProperty(Property property) { 
		m_properties.add(property);
	}
```
의미 없는 주석을 제거함으로써 코드가 한눈에 들어온다.  
변수 2개에 메소드가 1개인 클래스라는 사실이 드러난다.  

### 수직 거리 <a name = "7"></a>
__서로 밀집한 개념은 세로로 가까이 둬야 한다.__  
물론 두 개념이 서로 다른 파일에 속한다면 규칙이 통하진 않는다.  
하지만 서로 밀접한 개념은 한 파일에 속해야 마땅하다.(protected 변수를 피해야 하는 이유)  
같은 파일에 속할 정도로 밀접한 두 개념은 세로 거리로 연관성을 표현한다.  
연관성이란 한 개념을 이해하는 데 다른 개념이 중요한 정도이다.  
연관성이 깊은 두 개념이 멀리 떨어져 있으면 코드를 읽는 사람이 소스 파일과 클래스를 여기저기 뒤지게 된다.  

#### 변수 선언
변수는 사용하는 위치에 최대한 가까이 선언한다.  
```java
// InputStream이 함수 맨 처음에 선언되어있다.

private static void readPreferences() {
	InputStream is = null;
	try {
		is = new FileInputStream(getPreferencesFile()); 
		setPreferences(new Properties(getPreferences())); 
		getPreferences().load(is);
	} catch (IOException e) { 
		try {
			if (is != null) 
				is.close();
		} catch (IOException e1) {
		} 
	}
}

// 루프 제어 변수는 루프 문 내부에 선언
public int countTestCases() { 
	int count = 0;
	for (Test each : tests)
		count += each.countTestCases(); 
	return count;
}
```

#### 인스턴스 변수
인스턴스 변수는 클래스 맨 처음에 선언한다.  
변수 간에 세로로 거리를 두지 않는다.
```java
// bad
public class TestSuite implements Test {
	static public Test createTest(Class<? extends TestCase> theClass, String name) {
		... 
	}

	public static Constructor<? extends TestCase> 
	getTestConstructor(Class<? extends TestCase> theClass) 
	throws NoSuchMethodException {
		... 
	}

	public static Test warning(final String message) { 
		...
	}
	
	private static String exceptionToString(Throwable t) { 
		...
	}
	
	private String fName;

	private Vector<Test> fTests= new Vector<Test>(10);

	public TestSuite() { }
	
	public TestSuite(final Class<? extends TestCase> theClass) { 
		...
	}

	public TestSuite(Class<? extends TestCase> theClass, String name) { 
		...
	}
	
	... ... ... ... ...
}
```
위 코드에는 중간에 변수가 선언되어 있다.  
매우 찾기 힘들다.

#### 종속 함수
한 함수가 다른 함수를 호출한다면 두 함수는 세로로 가까이 배치한다.  
또한, 가능하다면 호출하는 함수를 호출되는 함수보다 먼저 배치한다.  
그래야 호출되는 함수를 찾기가 쉬워지며 전체 가독성도 높아진다.  
```java
public class WikiPageResponder implements SecureResponder { 
    ...
	
	public Response makeResponse(FitNesseContext context, Request request) throws Exception {
		String pageName = getPageNameOrDefault(request, "FrontPage");
		loadPage(pageName, context); 
		if (page == null)
			return notFoundResponse(context, request); 
		else
			return makePageResponse(context); 
		}

	private String getPageNameOrDefault(Request request, String defaultPageName) {
		String pageName = request.getResource(); 
		if (StringUtil.isBlank(pageName))
			pageName = defaultPageName;

		return pageName; 
	}
    ...
}
```

#### 개념적 유사성
친화도가 높을수록 코드를 가까이 배치한다.  
친화도가 높은 요인은 여러 가지다.
+ 한 함수가 다른 함수를 호출하는 경우
+ 변수와 그 변수를 사용하는 함수
+ 비슷한 동작을 수행하는 일군의 함수

```java
// 같은 assert 관련된 동작들을 수행하며, 명명법이 똑같고 기본 기능이 유사한 함수들로써 개념적 친화도가 높다.
// 이런 경우에는 종속성은 오히려 부차적 요인이므로, 종속적인 관계가 없더라도 가까이 배치하면 좋다.

public class Assert {
	static public void assertTrue(String message, boolean condition) {
		if (!condition) 
			fail(message);
	}

	static public void assertTrue(boolean condition) { 
		assertTrue(null, condition);
	}

	static public void assertFalse(String message, boolean condition) { 
		assertTrue(message, !condition);
	}
	
	static public void assertFalse(boolean condition) { 
		assertFalse(null, condition);
	} 
    ...
}
```

#### 세로 순서
일반적으로 함수 호출 종속성은 아래 방향으로 유지하므로, 호출되는 함수를 호출하는 함수보다 뒤에 배치한다.  
그러면 소스 코드가 자연스럽게 고차원에서 저차원으로 내려간다.  
즉, 가장 중요한 개념을 가장 먼저 표현하고, 세세한 사항은 뒤쪽에 표현되게 된다.  

<br>

## 가로 형식 맞추기 <a name = "8"></a>
저자가 프로젝트를 조사해본 결과 프로젝트의 행의 분포 길이는 규칙적이고, 프로그래머는 명백하게 짧은 행을 선호한다.  
__행의 길이는 100 ~ 120자를 권장하고 그 이상은 주의 부족이다.__  

### 가로 공백과 밀집도 <a name = "9"></a>
가로 공백은 밀접한 개념과 느슨한 개념을 표현한다.
```java
private void measureLine(String line) { 
	lineCount++;
	
	
	int lineSize = line.length();
	totalChars += lineSize; 
	
	lineWidthHistogram.addLine(lineSize, lineCount);
	recordWidestLine(lineSize);
}
```
할당 연산자 좌우로 공백을 주어 왼쪽, 오른쪽 요소가 확실하게 구분된다.  
반면 함수 이름과 괄호 사이에는 공백을 없앰으로써 함수와 인수의 밀접함을 보여준다.  
괄호 안의 인수끼리는 쉼표 뒤의 공백을 통해 인수가 별개라는 사실을 보여준다.  

### 가로 정렬 <a name = "10"></a>
```java
public class FitNesseExpediter implements ResponseSender {
	private		Socket		  socket;
	private 	InputStream 	  input;
	private 	OutputStream 	  output;
	private 	Reques		  request; 		
	private 	Response 	  response;	
	private 	FitNesseContex	  context; 
	protected 	long		  requestParsingTimeLimit;
	private 	long		  requestProgress;
	private 	long		  requestParsingDeadline;
	private 	boolean		  hasError;
	
	... 
```
보기엔 깔끔해 보일지 모르나, 코드가 엉뚱한 부분을 강조해 변수 유형을 자연스레 무시하고 이름부터 읽게 된다.  
게다가 코드 형식을 자동으로 맞춰주는 도구는 대다수가 위와 같은 정렬을 무시하고 원래대로 되돌린다.  
그러므로 선언문과 할당문을 별도로 정렬할 필요가 없다.  
정렬이 필요할 정도로 목록이 길다면 목록의 길이가 문제이지 정렬이 부족해서가 아니다.  
선언부가 길다는 것은 클래스를 쪼개야 한다는 것을 의미한다.  

### 들여쓰기 <a name = "11"></a>
들여쓰기한 파일은 구조가 한눈에 들어온다.  
```java
// bad
public class CommentWidget extends TextWidget {
	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
	
	public CommentWidget(ParentWidget parent, String text){super(parent, text);}
	public String render() throws Exception {return ""; } 
}
```
```java
// good
public class CommentWidget extends TextWidget {
	public static final String REGEXP = "^#[^\r\n]*(?:(?:\r\n)|\n|\r)?";
	
	public CommentWidget(ParentWidget parent, String text){
		super(parent, text);
	}
	
	public String render() throws Exception {
		return ""; 
	} 
}
```

### 가짜 범위 <a name = "12"></a>
빈 while문이나 for문을 접할 때가 있다.  
가능한 피해야하지만 피하지 못할 때는 빈 블록을 올바로 들여쓰고 괄호로 감싼다.  
세미콜론은 새 행에다가 제대로 들여써서 넣어준다.(이렇게 하지 않으면 눈에 띄지 않는다.)  
```java
while(dis.read(buf, 0, readBufferSize) != -1)
;
```
<Br>

## 팀 규칙 <a name = "13"></a>
프로그래머라면 각자 선호하는 규칙이 있더라도 팀에 속한다면 자신이 선호해야 할 규칙은 바로 팀 규칙이다.  
모든 팀원이 규칙을 따라야 소프트웨어가 일관적인 스타일을 보인다.  
좋은 소프트웨어 시스템은 읽기 쉬운 문서로 이뤄지고, 스타일은 일관적이고 매끄러워야 한다는 사실을 기억하라.  

<br>

## 밥 아저씨의 형식 규칙 <a name = "14"></a>
아래 코드는 저자가 사용하는 규칙이 들어나는 코드이다.
```java
public class CodeAnalyzer implements JavaFileAnalysis { 
	private int lineCount;
	private int maxLineWidth;
	private int widestLineNumber;
	private LineWidthHistogram lineWidthHistogram; 
	private int totalChars;
	
	public CodeAnalyzer() {
		lineWidthHistogram = new LineWidthHistogram();
	}
	
	public static List<File> findJavaFiles(File parentDirectory) { 
		List<File> files = new ArrayList<File>(); 
		findJavaFiles(parentDirectory, files);
		return files;
	}
	
	private static void findJavaFiles(File parentDirectory, List<File> files) {
		for (File file : parentDirectory.listFiles()) {
			if (file.getName().endsWith(".java")) 
				files.add(file);
			else if (file.isDirectory()) 
				findJavaFiles(file, files);
		} 
	}
	
	public void analyzeFile(File javaFile) throws Exception { 
		BufferedReader br = new BufferedReader(new FileReader(javaFile)); 
		String line;
		while ((line = br.readLine()) != null)
			measureLine(line); 
	}
	
	private void measureLine(String line) { 
		lineCount++;
		int lineSize = line.length();
		totalChars += lineSize; 
		lineWidthHistogram.addLine(lineSize, lineCount);
		recordWidestLine(lineSize);
	}
	
	private void recordWidestLine(int lineSize) { 
		if (lineSize > maxLineWidth) {
			maxLineWidth = lineSize;
			widestLineNumber = lineCount; 
		}
	}

	public int getLineCount() { 
		return lineCount;
	}

	public int getMaxLineWidth() { 
		return maxLineWidth;
	}

	public int getWidestLineNumber() { 
		return widestLineNumber;
	}

	public LineWidthHistogram getLineWidthHistogram() {
		return lineWidthHistogram;
	}
	
	public double getMeanLineWidth() { 
		return (double)totalChars/lineCount;
	}

	public int getMedianLineWidth() {
		Integer[] sortedWidths = getSortedWidths(); 
		int cumulativeLineCount = 0;
		for (int width : sortedWidths) {
			cumulativeLineCount += lineCountForWidth(width); 
			if (cumulativeLineCount > lineCount/2)
				return width;
		}
		throw new Error("Cannot get here"); 
	}
	
	private int lineCountForWidth(int width) {
		return lineWidthHistogram.getLinesforWidth(width).size();
	}
	
	private Integer[] getSortedWidths() {
		Set<Integer> widths = lineWidthHistogram.getWidths(); 
		Integer[] sortedWidths = (widths.toArray(new Integer[0])); 
		Arrays.sort(sortedWidths);
		return sortedWidths;
	} 
}
```