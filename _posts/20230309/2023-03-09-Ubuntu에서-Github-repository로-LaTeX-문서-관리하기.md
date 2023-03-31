---
title: Ubuntu에서 Github repository로 LaTeX 문서 관리하기
# date: 2022-11-05 13:30:00 +/-TTTT
categories: [etc.]
tags: [Ubuntu, Github, LaTeX]
---

우분투 환경에서 LaTeX 문서를 작성하고 관리하기 편리한 방법을 소개한다.

## texlive 설치
```bash
sudo apt install texlive-full
```

## Github repository 생성
레포지토리 생성 화면으로 들어가서, 다음과 같이 설정하여 생성한다.

* 공개 범위: Private
* Add .gitignore: 템플릿을 TeX 으로 설정

![gitignore](/assets/img/20230309/gitignore.png){: .shadow }

생성 후 레포지토리에는 다음과 같이 `.gitignore` 파일 하나만 만들어진다.

![ieee-paper repo](/assets/img/20230309/ieee-paper-repo.png){: .shadow }

> 이 파일은 문서 작업 후 렌더링으로 만들어지는 파일을 Git 버전 관리 시에 반영하지 않도록 한다.
만약 렌더링 결과물인 pdf 파일도 반영되지 않게 하려면, `.gitignore` 파일을 열고 주석처리된 `# *.pdf` 부분을 주석 해제한다.
{: .prompt-info }

만들어진 레포지토리를 로컬 환경에서 작업할 수 있도록 `git clone` 한다.

```bash
git clone {레포지토리 주소}
```

## LaTeX 템플릿 다운로드
IEEE 논문 템플릿이 필요하다면 다음 링크에서 다운로드하면 된다.

* [IEEE Template Selector](https://template-selector.ieee.org/secure/templateSelector/publicationType)

필요한 저널 또는 학회의 LaTeX 템플릿을 선택하면 zip 파일이 다운로드된다.
압축을 풀고 로컬 레포지토리 경로 내에 복사-붙여넣기 한다.

![template-downloaded](/assets/img/20230309/template-downloaded.png){: .shadow }
_붙여넣기 후 VS Code에서 확인한 모습_

## 문서 작성 (VS Code)
편집기로 VS Code 를 사용한다면 편집과 미리보기를 편리하게 할 수 있다.
Extension 중에서 LaTeX Workshop 을 설치한다.

![latex-workshop](/assets/img/20230309/latex-workshop.png){: .shadow }
_'LaTeX Workshop' Extension 을 설치한 모습_

설치 후 편집기에서 문서 파일을 열면 우측 상단에 빌드 버튼이 활성화된다.

![before-build](/assets/img/20230309/before-build.png){: .shadow }

버튼을 누르면 빌드가 진행되면서 pdf 파일이 생성된다.
다음과 같이 pdf 파일을 VS Code 에서 열어볼 수도 있다.

![overall-screen](/assets/img/20230309/overall-screen.png){: .shadow }

> pdf 미리보기 화면에서, 편집하고 싶은 부분을 Ctrl 키 입력과 함께 좌클릭하면 해당 부분을 곧바로 편집할 수 있도록 tex 파일이 열린다.
{: .prompt-tip }
