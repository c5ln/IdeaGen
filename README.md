# IdeaGen
github에서 idea 찾기 프로젝트.

1. github에서 stars 1000개 이상 데이터를 긁어온다.
2. 데이터에서 무작위 2개를 묶어서, 새로운 아이디어를 만든다.(Meta-Llama-3.1-8B 이용) 
3. 새로 생성된 아이디어에 biz, tech 점수를 부여하고, 프로젝트에 대한 설명을 작성한다.(Meta-Llama-3.1-8B 이용) 
4. SQLite를 이용해서 저장한다.

### GitHub에서 레포 수집
"""python main.py collect              
python main.py collect -p 10       

### 아이디어 생성 + 평가
python main.py run
python main.py run -n 5             # 5개만 생성

### 결과 조회
python main.py results
python main.py results --min-biz 7  # biz_score 7 이상
python main.py results --min-tech 8 --limit 5

### 불러온 repo 보기 
python main.py repos

### 삭제 명령어
python main.py delete-repos --all   
python main.py delete-repos --ids 1,3,5
python main.py delete-repos --name awesome (awesome 이름이 포함된 repo 삭제)
python main.py delete-results --all
python main.py delete-results --ids 1,3


### 1차 시도 결과 
문제 1.
  GitHub Star 상위권에는 awesome-list, free-programming-books, interview-guide 같은 지식 모음집이 많다.
  
  현상(실제 사례):
      Node.js + CS-Notes = Node.js 지식 그래프 (#6)
      Interview Guide + Free Domains = 도메인 특화 코딩 학습 사이트 (#5)
      Java Guide + Node Best Practices = 시스템 디자인 가이드 (#9)  
  한계: 이런 조합은 좀 아쉽다.

문제 2.
  Biz 점수가 너무 높다. 10개 뽑았는데, 모두 8점을 넘었다.

해결책:
① 비코드 리포지토리 제외하기 : GitHub API에서 리포지토리를 가져올 때, Language가 None이거나 Markdown인 경우를 필터링하거나, 토픽에 awesome-list, books가 있으면 제외하는 로직을 추가한다.
② 점수 기준 강화 

### 2차 시도 결과
문제 1. 아이디어 생성까지 기다리는 시간이 오래 걸린다.
  현상 : 아이디어를 다 만들 때까지 기다린다. 

문제 2. LLM이 가끔 형식을 벗어난 응답을 준다. 
  현상 : score가 범위를 벗어난다.


해결책 :
① SSE (Server-Sent Events) 스트리밍: Flask의 yield와 JS의 EventSource를 사용하여, LLM이 한 글자씩 생성할 때마다 실시간으로 웹 UI에 텍스트가 흐르도록 변경
② GBNF (Grammar-Based Normalization) 적용 : llama-cpp-python은 GBNF (Grammar)를 지원함. JSON 스키마를 GBNF 문법으로 정의하여 주입하면, 모델이 문법에 어긋나는 토큰 자체를 생성하지 못하게 강제할 수 있다.
