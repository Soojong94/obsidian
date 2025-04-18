---
{"dg-publish":true,"permalink":"/Tips/Naver Cloud Papago Translation 사용법/"}
---


## 참조 링크

- [Papago Translation 개요](https://guide-gov.ncloud-docs.com/docs/ko/papagotranslation-overview)
- [Papago Translation API](https://guide-gov.ncloud-docs.com/docs/papagotranslation-api)
- [translation (웹사이트 번역요청 가이드)](https://api-gov.ncloud-docs.com/docs/ai-naver-papagowebsitetranslation-translation)

## 1. 개요

네이버 파파고 API는 크게 4가지 서비스를 제공하지만, 여기서는 2가지만 다룹니다.

- Text Translation: 텍스트 단위 번역
- Website Translation: 웹페이지 HTML 전체 번역

## 2. API 연동 기본 설정

### 인증 설정

Headers 설정:

```
Content-Type: application/x-www-form-urlencoded
X-NCP-APIGW-API-KEY-ID: [발급받은 Client ID]
X-NCP-APIGW-API-KEY: [발급받은 Client Secret]
```

### 기본 요청 형식

- HTTP Method: POST
- Endpoint:
    - 텍스트 번역: https://naveropenapi.apigw.gov-ntruss.com/nmt/v1/translation
    - 웹사이트 번역: https://naveropenapi.apigw.gov-ntruss.com/web-trans/v1/translate

### Postman 을 이용한 API 응답 테스트

![Pasted image 20250418132653.png](/img/user/images/Pasted%20image%2020250418132653.png)

![Pasted image 20250418132714.png](/img/user/images/Pasted%20image%2020250418132714.png)


## 3. 비용 최적화 전략

### Text Translation과 Website Translation 혼합 사용

- 시스템 특성에 맞게 두 서비스를 혼용하여 비용 최적화 가능
- 전체 웹페이지가 아닌 필요한 텍스트 부분만 선별적으로 번역 처리

### 번역 대상 최소화 방법

```javascript
// 번역이 필요한 댓글 요소들만 선택
const comments = document.getElementsByClassName('comment_item_wrapper');

// 선택된 요소들을 배열로 변환하여 HTML 추출
const htmlsToRequest = Array.from(comments).map(comment => comment.outerHTML);
```

## 4. UI 안정성 유지 방안

### 레이아웃 유지 기법

- DOM 구조를 유지하면서 번역이 필요한 콘텐츠 영역만 선택적으로 처리

### 동적 UI 요소 처리

```javascript
// 드롭다운 번역 예시
function translateDropdown(selectElement, sourceLanguage, targetLanguage) {
  // 드롭다운의 원본 옵션 값들을 백업 (기능 유지 목적)
  const texts = Array.from(selectElement.options).map(option => option.text);
  
  // 서버에 번역 요청
  translateTexts(texts, sourceLanguage, targetLanguage)
    .then(translatedTexts => {
      // 번역된 텍스트로 드롭다운 옵션 업데이트 (value 값은 유지)
      translatedTexts.forEach((text, index) => {
        selectElement.options[index].text = text;
      });
    });
}
```

## 5. 서버 측 프록시 구현

```javascript
// Node.js + Express 서버 예시
const express = require('express');
const axios = require('axios');
const app = express();

app.use(express.json());

// 텍스트 번역 API 엔드포인트
app.post('/api/translate/text', async (req, res) => {
  try {
    const { source, target, text } = req.body;
    
    // API 요청 데이터 준비
    const formData = new URLSearchParams();
    formData.append('source', source);
    formData.append('target', target);
    formData.append('text', text);
    
    // 네이버 파파고 API 호출
    const response = await axios.post(
      'https://naveropenapi.apigw.gov-ntruss.com/nmt/v1/translation',
      formData,
      {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'X-NCP-APIGW-API-KEY-ID': process.env.NAVER_CLIENT_ID,
          'X-NCP-APIGW-API-KEY': process.env.NAVER_CLIENT_SECRET
        }
      }
    );
    
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// 웹사이트 번역 API 엔드포인트
app.post('/api/translate/website', async (req, res) => {
  try {
    const { source, target, html } = req.body;
    
    const formData = new URLSearchParams();
    formData.append('source', source);
    formData.append('target', target);
    formData.append('html', html);
    
    const response = await axios.post(
      'https://naveropenapi.apigw.gov-ntruss.com/web-trans/v1/translate',
      formData,
      {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          'X-NCP-APIGW-API-KEY-ID': process.env.NAVER_CLIENT_ID,
          'X-NCP-APIGW-API-KEY': process.env.NAVER_CLIENT_SECRET
        }
      }
    );
    
    res.json(response.data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

app.listen(3000, () => {
  console.log('Translation proxy server running on port 3000');
});
```

## 6. 클라이언트 측 구현

```javascript
// 텍스트 번역 함수
async function translateText(text, source, target) {
  try {
    const response = await fetch('/api/translate/text', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ source, target, text })
    });
    
    const data = await response.json();
    return data.message.result.translatedText;
  } catch (error) {
    console.error('번역 오류:', error);
    return text; // 오류 시 원본 반환
  }
}

// HTML 번역 함수
async function translateHtml(html, source, target) {
  try {
    const response = await fetch('/api/translate/website', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ source, target, html })
    });
    
    const data = await response.json();
    return data.message.result.translatedHtml;
  } catch (error) {
    console.error('HTML 번역 오류:', error);
    return html; // 오류 시 원본 반환
  }
}
```

## 7. 주요 고려사항 및 해결 방안

### 보안 및 접근성

브라우저에서 직접 API 호출 불가 이유:

- CORS 정책: 파파고 API는 CORS 옵션 미제공
- API 키 노출 위험: 클라이언트 측 호출 시 키 탈취 가능성

서버 측에서 중계(Proxy) 방식으로 구현 필요

### UI 안정성

- 단순 텍스트 분할이 아닌 태그 구조를 유지하며 번역 요청
- 번역 대상 요소를 정확히 식별하여 처리

### 성능 최적화

- 번역이 필요한 요소만 선별적으로 처리
- 동적 콘텐츠는 필요 시에만 번역 요청

## 8. 결론 및 권장사항

- 번역 대상을 최소화하여 비용 효율성 확보
- 서버 측 프록시를 통한 API 호출로 보안 강화
- DOM 구조를 유지하는 선별적 번역으로 UI 안정성 보장
- 번역이 필요한 동적 요소는 Text Translation API로 처리

## 특정 태그를 지정하여 번역하는 예시 코드

```javascript
/**
 * HTML 페이지에서 특정 태그/클래스를 가진 요소만 선택적으로 번역하는 예시
 */

// 1. 번역이 필요한 요소들 선택하기
function selectElementsToTranslate() {
  // 제목 요소 선택 (h1, h2, h3 태그)
  const headings = document.querySelectorAll('h1, h2, h3');
  
  // 본문 문단 요소 선택 (특정 클래스를 가진 p 태그)
  const paragraphs = document.querySelectorAll('p.translate-this');
  
  // 메뉴 아이템 선택 (특정 ID를 가진 nav 내의 li 태그)
  const menuItems = document.querySelectorAll('#main-menu li');
  
  // 모든 선택된 요소를 하나의 배열로 합치기
  return [...headings, ...paragraphs, ...menuItems];
}

// 2. 선택된 요소들의 텍스트 추출 및 번역 요청
async function translateSelectedElements() {
  // 번역할 요소들 선택
  const elements = selectElementsToTranslate();
  
  // 요소가 없으면 종료
  if (elements.length === 0) {
    console.log('번역할 요소가 없습니다.');
    return;
  }
  
  // 각 요소의 텍스트 내용과 요소 자체를 매핑
  const elementsWithText = elements.map(element => ({
    element: element,
    text: element.innerText.trim()
  })).filter(item => item.text); // 빈 텍스트 제외
  
  // 번역할 텍스트 목록
  const textsToTranslate = elementsWithText.map(item => item.text);
  
  try {
    // 서버에 번역 요청 (배열로 한 번에 요청)
    const response = await fetch('/api/translate/batch', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        source: 'ko',  // 원본 언어 (한국어)
        target: 'en',  // 타겟 언어 (영어)
        texts: textsToTranslate
      })
    });
    
    // 응답 처리
    const data = await response.json();
    const translatedTexts = data.translatedTexts || [];
    
    // 번역된 텍스트를 각 요소에 적용
    elementsWithText.forEach((item, index) => {
      if (translatedTexts[index]) {
        item.element.innerText = translatedTexts[index];
      }
    });
    
    console.log(`${translatedTexts.length}개 요소 번역 완료`);
  } catch (error) {
    console.error('번역 처리 중 오류 발생:', error);
  }
}

// 3. 번역 버튼 이벤트 리스너 등록
document.getElementById('translate-button').addEventListener('click', () => {
  translateSelectedElements();
});
```

## 서버 측 배치 번역 처리 엔드포인트 예시

```javascript
// Express 서버에서 배치 번역을 처리하는 엔드포인트
app.post('/api/translate/batch', async (req, res) => {
  try {
    const { source, target, texts } = req.body;
    
    if (!Array.isArray(texts) || texts.length === 0) {
      return res.status(400).json({ error: '번역할 텍스트 배열이 필요합니다.' });
    }
    
    // 결과를 저장할 배열
    const translatedTexts = [];
    
    // 각 텍스트마다 파파고 API 호출
    for (const text of texts) {
      // API 호출 데이터 준비
      const formData = new URLSearchParams();
      formData.append('source', source);
      formData.append('target', target);
      formData.append('text', text);
      
      // 파파고 API 호출
      const response = await axios.post(
        'https://naveropenapi.apigw.gov-ntruss.com/nmt/v1/translation',
        formData,
        {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
            'X-NCP-APIGW-API-KEY-ID': process.env.NAVER_CLIENT_ID,
            'X-NCP-APIGW-API-KEY': process.env.NAVER_CLIENT_SECRET
          }
        }
      );
      
      // 번역 결과 추출 및 저장
      const translatedText = response.data.message.result.translatedText;
      translatedTexts.push(translatedText);
    }
    
    // 모든 번역 결과 반환
    res.json({ translatedTexts });
  } catch (error) {
    console.error('배치 번역 오류:', error);
    res.status(500).json({ error: '번역 처리 중 오류가 발생했습니다.' });
  }
});
```