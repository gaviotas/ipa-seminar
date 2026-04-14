# Seminar Material Guidelines

이 문서는 앞으로 논문 세미나용 GitHub Pages 자료를 만들 때 잊지 말아야 할 사항을 정리한 체크리스트다.

## Goal
- 논문 원문을 따로 열지 않아도 세미나 자료만으로 핵심 내용을 전달할 수 있어야 한다.
- 비전공자도 따라올 수 있어야 하지만, 전공 박사 청중에게는 technical depth가 부족하지 않아야 한다.
- 읽기용 블로그가 아니라 발표자가 바로 사용할 수 있는 세미나 패키지여야 한다.

## Content Structure
- 논문 메타 정보부터 먼저 제시한다.
- 제목, 학회, 저자, affiliation, arXiv, 프로젝트 링크를 맨 앞에 배치한다.
- teaser figure 또는 프로젝트 대표 visual을 초반에 넣어 첫인상을 만든다.
- 문제 정의, 방법론, 평가, 결과, 한계, takeaways 순서가 자연스럽다.
- 각 섹션은 왜 필요한지 설명한 뒤 technical detail로 들어간다.
- 논문 핵심 기여는 초반에 3개 이하로 요약한다.

## Technical Depth
- 문제를 수식으로 명확히 정리한다.
- notation table을 제공해 기호 혼동을 줄인다.
- 핵심 알고리즘은 pseudocode 또는 step-by-step 절차로 다시 쓴다.
- 방법론 섹션에는 논문 식을 직접 포함한다.
- implementation detail도 빠뜨리지 않는다.
- layer index, coefficient, model backbone, metric definition 같은 값은 구체적으로 적는다.
- ablation과 failure mode를 반드시 다룬다.

## Results Presentation
- 표는 논문 원본 구조를 최대한 유지한다.
- family/group 정보는 column보다 group row가 더 읽기 좋을 수 있다.
- 결과표 뒤에는 반드시 해석 문단을 붙인다.
- absolute gain과 relative gain을 같이 적으면 전달력이 좋아진다.
- qualitative figure는 quantitative table 뒤에 배치하는 것이 자연스럽다.
- 성공 사례뿐 아니라 실패 사례 해석도 포함한다.

## Writing Style
- 요약은 짧고 강하게 쓴다.
- 본문은 설명형 문장으로 쓰되, 발표자가 읽기 쉬워야 한다.
- 섹션 제목만 봐도 전체 논리 흐름이 보여야 한다.
- bullet list는 남발하지 말고, 진짜 list일 때만 사용한다.
- 결과 해석과 비판적 논의는 분리한다.
- “논문 주장”과 “내 해석”을 섞지 말고 구분해 쓴다.

## Visual / Design
- 과한 장식보다 고급 미니멀 스타일이 세미나 자료에 잘 맞는다.
- 타이포가 가장 중요하다.
- 제목, 본문, 캡션, 메타 텍스트의 크기와 색 대비를 분리한다.
- 섹션 간 구분이 스크롤 중에도 분명해야 한다.
- 표, figure, pseudocode 박스의 spacing을 통일한다.
- 모바일에서도 깨지지 않아야 한다.

## Math / Rendering
- 수식 렌더링은 반드시 확인한다.
- MathJax 또는 KaTeX 설정을 미리 넣는다.
- inline math와 display math가 모두 정상인지 확인한다.
- table 안 수식도 렌더링되는지 확인한다.
- code block 안 수식은 기본적으로 렌더링되지 않을 수 있으므로 주의한다.
- pseudocode 안 수식이 필요하면 code fence 대신 MathJax-friendly markup을 사용한다.

## Media Usage
- 논문 figure는 적절한 위치에 배치한다.
- teaser는 초반, method figure는 방법론, qualitative figure는 결과 뒤가 좋다.
- 영상/애니메이션은 정말 도움이 될 때만 넣는다.
- 잘못된 embed가 들어가지 않도록 source를 검증한다.
- 프로젝트 페이지 media를 가져올 때 링크 안정성을 꼭 확인한다.

## Presentation Notes
- 포스트와 별도로 발표 노트를 유지한다.
- 슬라이드별 제목, 핵심 문장, 말할 멘트를 적어둔다.
- 15분 기준 시간 배분을 적는다.
- 예상 Q&A를 미리 적어둔다.
- 백업 슬라이드 아이디어도 같이 남겨둔다.

## Verification Checklist
- 로컬에서 Jekyll build가 성공하는지 확인
- 주요 링크가 404가 아닌지 확인
- GitHub Pages baseurl이 맞는지 확인
- 수식 렌더링 확인
- 표 레이아웃 확인
- figure 및 iframe/video 로딩 확인
- 모바일 폭에서 표와 카드가 깨지지 않는지 확인

## GitHub Pages / Deployment
- `main` 브랜치와 root 배포 여부 확인
- `_config.yml`의 `url`, `baseurl`을 저장소에 맞게 설정
- post URL이 baseurl을 타는지 확인
- 캐시 때문에 반영이 늦을 수 있으니 최종 확인 시 강력 새로고침

## Recommended Workflow
1. 논문 원문과 프로젝트 페이지를 먼저 읽고 구조를 잡는다.
2. 포스트 초안을 쓴다.
3. 발표 노트를 따로 작성한다.
4. figure/table/math를 붙인다.
5. 스타일을 손본다.
6. 로컬 빌드로 검증한다.
7. push 후 Pages에서 다시 확인한다.

## Common Failure Cases
- 표가 깨짐
- baseurl 누락으로 링크 404
- MathJax 미적용
- code fence 안 수식 미렌더링
- 프로젝트 페이지의 placeholder video를 실제 demo로 착각
- 디자인을 과하게 만져서 오히려 산만해짐

## Principle To Remember
- 세미나 자료는 “예쁜 블로그”보다 “논문을 대체할 수 있는 설명 가능한 문서”에 가까워야 한다.
