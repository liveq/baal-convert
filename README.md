# #14 이미지 변환기

**URL:** convert.baal.co.kr

## 서비스 내용

이미지 포맷 변환 (JPEG/PNG/WebP/AVIF). 최신 포맷 지원. 일괄 변환. 품질 조절 가능

## 기능 요구사항

- [ ] 이미지 업로드 (드래그 앤 드롭, 다중 파일)
- [ ] 포맷 지원:
  - [ ] JPEG (입력/출력)
  - [ ] PNG (입력/출력)
  - [ ] WebP (입력/출력)
  - [ ] AVIF (입력/출력)
  - [ ] GIF (입력만, 첫 프레임 추출)
  - [ ] BMP (입력만)
- [ ] 품질 조절 슬라이더 (1-100%)
- [ ] 투명 배경 유지 (PNG → PNG, PNG → WebP)
- [ ] 일괄 변환 (최대 20개 파일)
- [ ] 전/후 용량 비교
- [ ] 개별 다운로드
- [ ] ZIP 일괄 다운로드
- [ ] 포맷별 최적 설정 프리셋

## 경쟁사 분석 (2025년 기준)

### 인기 사이트 TOP 5

1. **CloudConvert** - 가장 강력한 변환 서비스
   - 강점: 200+ 포맷 지원, API 제공, 고품질
   - 약점: 무료는 하루 25회 제한, 회원가입 필요

2. **Convertio** - 웹 기반 변환 도구
   - 강점: 다양한 포맷, 깔끔한 UI
   - 약점: 무료는 100MB 제한, 광고 많음

3. **Online-Convert** - 올인원 변환 서비스
   - 강점: 이미지 외 비디오/오디오/문서 지원
   - 약점: 속도 느림, 복잡한 UI

4. **FreeConvert** - 무료 변환 서비스
   - 강점: 무료, 제한 적음, 다양한 옵션
   - 약점: 광고 많음, 대기 시간

5. **iLoveIMG** - 이미지 도구 통합
   - 강점: 변환 외 다양한 이미지 기능
   - 약점: WebP/AVIF 지원 제한적

### 우리의 차별화 전략

- ✅ **최신 포맷 지원** - WebP, AVIF (2025년 표준)
- ✅ **브라우저 기반** - 서버 업로드 불필요, 프라이버시
- ✅ **완전 무료** - 횟수/용량 제한 없음
- ✅ **빠른 처리** - 로컬 변환, 즉시 다운로드
- ✅ **품질 조절** - 포맷별 세부 설정
- ✅ **일괄 변환** - ZIP 다운로드
- ✅ **다크모드** 지원
- ✅ **통합 탭** - 압축/리사이즈/업스케일과 연결

## 주요 라이브러리

### 옵션 1: Canvas API + browser-image-compression (추천!)

Canvas API로 포맷 변환, browser-image-compression으로 품질 최적화

```html
<script src="https://cdn.jsdelivr.net/npm/browser-image-compression@2.0.2/dist/browser-image-compression.js"></script>
```

```javascript
// 1. 이미지 로드
const img = new Image();
img.src = URL.createObjectURL(file);
await img.decode();

// 2. Canvas에 그리기
const canvas = document.createElement('canvas');
canvas.width = img.width;
canvas.height = img.height;
const ctx = canvas.getContext('2d', { alpha: true }); // 투명도 지원

// PNG 투명 배경 유지
if (outputFormat === 'image/png' || outputFormat === 'image/webp') {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
}

ctx.drawImage(img, 0, 0);

// 3. 포맷 변환
const blob = await new Promise((resolve) => {
  canvas.toBlob(resolve, outputFormat, quality); // 'image/jpeg', 'image/png', 'image/webp'
});

// 4. browser-image-compression으로 최적화 (선택적)
if (needCompression) {
  const file = new File([blob], 'temp', { type: outputFormat });
  const compressedBlob = await imageCompression(file, {
    maxSizeMB: 1,
    useWebWorker: true,
    fileType: outputFormat,
    initialQuality: quality / 100
  });
  return compressedBlob;
}

return blob;
```

**포맷별 설정:**

```javascript
const formatConfigs = {
  'image/jpeg': {
    quality: 0.9,
    extension: 'jpg',
    supportsTransparency: false
  },
  'image/png': {
    quality: 1.0, // PNG는 무손실
    extension: 'png',
    supportsTransparency: true
  },
  'image/webp': {
    quality: 0.9,
    extension: 'webp',
    supportsTransparency: true
  }
};

// 사용 예
const config = formatConfigs[outputFormat];
canvas.toBlob((blob) => {
  downloadFile(blob, `converted.${config.extension}`);
}, outputFormat, config.quality);
```

**특징:**
- ✅ 빠름 (1-2초)
- ✅ 모든 브라우저 지원 (WebP는 최신 브라우저만)
- ✅ 투명도 유지 (PNG, WebP)
- ✅ 품질 조절 가능
- ⚠️ AVIF는 지원 제한적 (2025년 기준 80% 브라우저)

### 옵션 2: wasm-image-compressor (AVIF 완벽 지원)

WebAssembly 기반 고급 변환 (AVIF 포함)

```javascript
// AVIF 변환 (최신 브라우저)
async function convertToAVIF(file, quality = 0.8) {
  // HTML Canvas API는 AVIF를 완벽히 지원하지 않으므로
  // 폴백 전략 필요

  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');

  const img = new Image();
  img.src = URL.createObjectURL(file);
  await img.decode();

  canvas.width = img.width;
  canvas.height = img.height;
  ctx.drawImage(img, 0, 0);

  // AVIF 지원 체크
  const avifSupported = await checkAVIFSupport();

  if (avifSupported) {
    const blob = await new Promise((resolve) => {
      canvas.toBlob(resolve, 'image/avif', quality);
    });
    return blob;
  } else {
    // 폴백: WebP로 변환
    showToast('AVIF는 이 브라우저에서 지원되지 않습니다. WebP로 변환합니다.', 'warning');
    const blob = await new Promise((resolve) => {
      canvas.toBlob(resolve, 'image/webp', quality);
    });
    return blob;
  }
}

// AVIF 지원 체크 함수
async function checkAVIFSupport() {
  const avifTestImage = 'data:image/avif;base64,AAAAIGZ0eXBhdmlmAAAAAGF2aWZtaWYxbWlhZk1BMUIAAADybWV0YQAAAAAAAAAoaGRscgAAAAAAAAAAcGljdAAAAAAAAAAAAAAAAGxpYmF2aWYAAAAADnBpdG0AAAAAAAEAAAAeaWxvYwAAAABEAAABAAEAAAABAAABGgAAAB0AAAAoaWluZgAAAAAAAQAAABppbmZlAgAAAAABAABhdjAxQ29sb3IAAAAAamlwcnAAAABLaXBjbwAAABRpc3BlAAAAAAAAAAIAAAACAAAAEHBpeGkAAAAAAwgICAAAAAxhdjFDgQ0MAAAAABNjb2xybmNseAACAAIAAYAAAAAXaXBtYQAAAAAAAAABAAEEAQKDBAAAACVtZGF0EgAKCBgANogQEAwgMg8f8D///8WfhwB8+ErK42A=';

  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => resolve(true);
    img.onerror = () => resolve(false);
    img.src = avifTestImage;
  });
}
```

**AVIF 특징:**
- ✅ 최고 압축률 (JPEG 대비 50% 작음)
- ✅ 투명도 지원
- ✅ 품질 우수
- ⚠️ 브라우저 지원: Chrome 85+, Firefox 93+, Safari 16.1+ (2025년 약 80%)
- ⚠️ 변환 속도 느림 (3-5초)

## UI/UX 디자인 패턴

### 화면 구성

```
┌─────────────────────────────────┐
│  이미지 변환기                    │
│  최신 포맷(WebP, AVIF) 지원       │
├─────────────────────────────────┤
│  변환 설정:                       │
│  출력 포맷: [JPEG] [PNG] [WebP] [AVIF] │
│  품질: [━━━━━●──] 90%            │
│                                 │
│  ⚙️ 고급 옵션                     │
│  ☑ 투명 배경 유지 (PNG, WebP만)   │
│  ☑ 용량 최적화 (추가 압축)         │
├─────────────────────────────────┤
│  🖼️ 드래그 앤 드롭 영역            │
│  또는 클릭하여 파일 선택 (최대 20개)│
│                                 │
│  지원 포맷: JPEG, PNG, WebP, AVIF, GIF, BMP │
├─────────────────────────────────┤
│  변환 결과:                       │
│  📄 photo1.jpg → photo1.webp    │
│     2.5MB → 1.2MB (52% 감소)    │
│     [다운로드]                   │
│                                 │
│  📄 photo2.png → photo2.avif    │
│     1.8MB → 650KB (64% 감소)    │
│     [다운로드]                   │
├─────────────────────────────────┤
│  [전체 ZIP 다운로드]              │
└─────────────────────────────────┘
```

### 포맷별 추천 설정 (프리셋)

**웹 최적화 (WebP):**
- 포맷: WebP
- 품질: 85%
- 투명도: 유지
- 용도: 웹사이트 이미지

**최대 압축 (AVIF):**
- 포맷: AVIF
- 품질: 80%
- 투명도: 유지
- 용도: 대역폭 절약

**호환성 우선 (JPEG):**
- 포맷: JPEG
- 품질: 90%
- 투명도: 불가 (흰색 배경)
- 용도: 구형 시스템 지원

**무손실 (PNG):**
- 포맷: PNG
- 품질: 100%
- 투명도: 유지
- 용도: 로고, 아이콘

### 주요 기능 순서

1. 출력 포맷 선택 (JPEG/PNG/WebP/AVIF)
2. 품질 설정 (슬라이더)
3. 파일 업로드 (드래그 또는 클릭, 최대 20개)
4. 자동 변환 시작 (진행률 표시)
5. 전/후 용량 비교
6. 다운로드 (개별 또는 ZIP)

## 난이도 & 예상 기간

- **난이도:** 보통 (Canvas API, 포맷별 처리)
- **예상 기간:** 2일
- **실제 기간:** (작업 후 기록)

## 개발 일정

- [ ] Day 1 오전: UI 구성, 포맷 선택, 파일 업로드
- [ ] Day 1 오후: Canvas API 변환 (JPEG, PNG, WebP), 품질 조절
- [ ] Day 2 오전: AVIF 지원, 브라우저 호환성 체크, 폴백 처리
- [ ] Day 2 오후: 일괄 변환, ZIP 다운로드, 최적화, 테스트

## 트래픽 예상

⭐⭐⭐ 높음 - "이미지 변환, webp 변환" 검색량 매우 높음

## SEO 키워드

- 이미지 변환
- 이미지 포맷 변환
- JPEG PNG 변환
- WebP 변환
- AVIF 변환
- 이미지 형식 변경
- 무료 이미지 변환기
- 일괄 이미지 변환

## 이슈 & 해결방안

### 실제 문제점 (경쟁사 분석 & 브라우저 호환성 기반)

1. **PNG 투명 배경이 JPEG로 변환 시 검은색/흰색으로 변환**
   - 원인: JPEG는 투명도 미지원, Canvas 기본 배경색
   - 해결: 투명 PNG → JPEG 변환 시 흰색 배경 추가
   - 해결: 사용자에게 경고 메시지 표시
   - 코드:
     ```javascript
     function convertWithBackground(canvas, format, quality) {
       if (format === 'image/jpeg') {
         // JPEG는 투명도 미지원 → 흰색 배경 추가
         const tempCanvas = document.createElement('canvas');
         const tempCtx = tempCanvas.getContext('2d');
         tempCanvas.width = canvas.width;
         tempCanvas.height = canvas.height;

         // 흰색 배경 채우기
         tempCtx.fillStyle = '#FFFFFF';
         tempCtx.fillRect(0, 0, tempCanvas.width, tempCanvas.height);

         // 원본 이미지 그리기
         tempCtx.drawImage(canvas, 0, 0);

         return new Promise((resolve) => {
           tempCanvas.toBlob(resolve, format, quality);
         });
       } else {
         // PNG, WebP는 투명도 유지
         return new Promise((resolve) => {
           canvas.toBlob(resolve, format, quality);
         });
       }
     }

     // 사용 시 경고
     if (hasTransparency(sourceImage) && outputFormat === 'image/jpeg') {
       showToast('투명 배경이 흰색으로 변환됩니다.', 'warning');
     }
     ```

2. **WebP/AVIF 브라우저 호환성 문제**
   - 원인: 구형 브라우저 미지원 (IE, Safari 구버전)
   - 해결: 브라우저 지원 체크 후 폴백
   - 해결: 지원 불가 시 대체 포맷 제안
   - 코드:
     ```javascript
     // WebP 지원 체크
     async function checkWebPSupport() {
       const testImage = 'data:image/webp;base64,UklGRiQAAABXRUJQVlA4IBgAAAAwAQCdASoBAAEAAwA0JaQAA3AA/vuUAAA=';
       return new Promise((resolve) => {
         const img = new Image();
         img.onload = () => resolve(true);
         img.onerror = () => resolve(false);
         img.src = testImage;
       });
     }

     // AVIF 지원 체크 (위 코드 참조)

     // UI 초기화 시
     async function initFormatSelector() {
       const webpSupported = await checkWebPSupport();
       const avifSupported = await checkAVIFSupport();

       if (!webpSupported) {
         document.getElementById('webp-option').disabled = true;
         document.getElementById('webp-option').title = '이 브라우저는 WebP를 지원하지 않습니다.';
       }

       if (!avifSupported) {
         document.getElementById('avif-option').disabled = true;
         document.getElementById('avif-option').title = '이 브라우저는 AVIF를 지원하지 않습니다.';
       }
     }
     ```

3. **GIF 애니메이션 손실 (첫 프레임만 추출)**
   - 원인: Canvas API는 GIF 애니메이션 미지원
   - 해결: GIF는 첫 프레임만 추출됨을 명시
   - 해결: 사용자에게 경고 표시
   - 코드:
     ```javascript
     function handleFileUpload(file) {
       if (file.type === 'image/gif') {
         showToast('GIF 파일은 첫 번째 프레임만 변환됩니다.', 'warning');
         // 애니메이션 보존이 필요하면 다른 도구 추천
       }

       // 일반 이미지로 처리 (첫 프레임 자동 추출)
       const img = new Image();
       img.src = URL.createObjectURL(file);
       return img;
     }
     ```

4. **대용량 이미지 변환 시 메모리 부족**
   - 원인: Canvas에서 고해상도 이미지 처리
   - 해결: 파일 크기 제한 (20MB)
   - 해결: 이미지 크기 제한 (8000x8000px)
   - 코드:
     ```javascript
     const MAX_FILE_SIZE = 20 * 1024 * 1024; // 20MB
     const MAX_DIMENSION = 8000;

     async function validateImage(file) {
       if (file.size > MAX_FILE_SIZE) {
         showToast('파일이 너무 큽니다. (최대 20MB)', 'error');
         return false;
       }

       const img = await loadImage(file);
       if (img.width > MAX_DIMENSION || img.height > MAX_DIMENSION) {
         showToast(`이미지 크기가 너무 큽니다. (최대 ${MAX_DIMENSION}x${MAX_DIMENSION})`, 'error');
         return false;
       }

       return true;
     }
     ```

5. **JPEG 품질 저하 (재압축)**
   - 원인: JPEG → JPEG 변환 시 품질 손실
   - 해결: 원본이 JPEG이고 출력도 JPEG이면 경고
   - 해결: 품질 슬라이더를 95-100% 권장
   - 코드:
     ```javascript
     function checkQualityLoss(inputFormat, outputFormat, quality) {
       if (inputFormat === 'image/jpeg' && outputFormat === 'image/jpeg') {
         if (quality < 0.95) {
           showToast('JPEG → JPEG 변환은 품질 손실이 발생할 수 있습니다. 품질을 95% 이상으로 설정하세요.', 'warning');
         }
       }
     }

     // 자동 품질 조정
     function getRecommendedQuality(inputFormat, outputFormat) {
       if (inputFormat === 'image/jpeg' && outputFormat === 'image/jpeg') {
         return 0.98; // 거의 무손실
       } else if (outputFormat === 'image/webp') {
         return 0.90; // WebP 최적
       } else if (outputFormat === 'image/avif') {
         return 0.85; // AVIF 최적
       } else if (outputFormat === 'image/png') {
         return 1.0; // PNG 무손실
       }
       return 0.90;
     }
     ```

6. **Canvas toBlob 에러 (특정 이미지에서 실패)**
   - 원인: CORS 문제, 이미지 손상
   - 해결: try-catch로 에러 핸들링
   - 해결: 재시도 로직
   - 코드:
     ```javascript
     async function convertImageSafe(canvas, format, quality, retries = 3) {
       for (let i = 0; i < retries; i++) {
         try {
           const blob = await new Promise((resolve, reject) => {
             canvas.toBlob((blob) => {
               if (blob) {
                 resolve(blob);
               } else {
                 reject(new Error('toBlob failed'));
               }
             }, format, quality);
           });
           return blob;
         } catch (error) {
           console.error(`변환 실패 (시도 ${i + 1}/${retries}):`, error);
           if (i === retries - 1) {
             throw new Error('이미지 변환에 실패했습니다. 다른 이미지를 시도하세요.');
           }
           // 재시도 전 대기
           await new Promise(resolve => setTimeout(resolve, 500));
         }
       }
     }
     ```

7. **일괄 변환 시 파일명 충돌**
   - 원인: 동일 이름 파일 여러 개
   - 해결: 파일명에 인덱스 추가
   - 코드:
     ```javascript
     function generateUniqueFilename(originalName, format, index) {
       const nameWithoutExt = originalName.replace(/\.[^/.]+$/, '');
       const extension = formatConfigs[format].extension;
       return `${nameWithoutExt}_${index}.${extension}`;
     }

     // 사용 예
     const results = [];
     files.forEach((file, index) => {
       const newName = generateUniqueFilename(file.name, outputFormat, index + 1);
       results.push({ name: newName, blob: convertedBlob });
     });
     ```

8. **WebP 변환 시 예상보다 용량 증가**
   - 원인: 특정 이미지는 WebP가 비효율적 (단순 그래픽)
   - 해결: 변환 전/후 비교해서 작은 것 선택
   - 코드:
     ```javascript
     async function convertWithSizeCheck(file, format, quality) {
       const convertedBlob = await convertImage(file, format, quality);

       // 원본보다 크면 원본 유지 (포맷만 변경 안함)
       if (convertedBlob.size > file.size) {
         showToast(`${file.name}: 변환 후 용량이 증가하여 원본을 유지합니다.`, 'info');
         return file;
       }

       return convertedBlob;
     }
     ```

## 통합 전략 (다른 이미지 도구와 연결)

### compress.baal.co.kr, resize.baal.co.kr, upscale.baal.co.kr

**탭 추가:**
```html
<div class="mode-tabs">
  <button class="tab" data-mode="compress">압축</button>
  <button class="tab" data-mode="resize">리사이즈</button>
  <button class="tab active" data-mode="convert">변환</button>
  <button class="tab" data-mode="upscale">업스케일</button>
</div>
```

**변환 탭 클릭 시:**
```javascript
if (mode === 'convert') {
  document.getElementById('tool-container').innerHTML = renderConvertUI();
  // 포맷 선택 표시 (JPEG/PNG/WebP/AVIF)
  // 품질 슬라이더 표시
}
```

**탭 간 데이터 공유:**
```javascript
// 압축 → 변환 워크플로우
function compressThenConvert(file) {
  // 1. 압축
  const compressed = await compressImage(file);

  // 2. 변환 탭으로 이동
  switchTab('convert');

  // 3. 압축된 파일을 변환 입력으로
  loadFileToConvert(compressed);
}

// 리사이즈 → 변환 워크플로우
function resizeThenConvert(file) {
  // 1. 리사이즈
  const resized = await resizeImage(file);

  // 2. 변환
  const converted = await convertImage(resized, 'image/webp', 0.9);

  return converted;
}
```

### convert.baal.co.kr

**메인 모드: 변환**
**탭: [변환] [압축] [리사이즈] [업스케일]**

## 추가 기능 (선택사항)

### 1. 메타데이터 제거 옵션

EXIF, GPS 정보 제거로 프라이버시 보호

```javascript
function removeMetadata(canvas, format, quality) {
  // Canvas는 자동으로 메타데이터 제거
  // toBlob 호출만으로 EXIF 정보가 사라짐
  return new Promise((resolve) => {
    canvas.toBlob(resolve, format, quality);
  });
}

// UI 체크박스
// ☑ 메타데이터 제거 (EXIF, GPS 정보)
```

### 2. 색 공간 변환

sRGB, Adobe RGB 변환

```javascript
ctx.filter = 'saturate(1.2)'; // 색상 조정
ctx.drawImage(img, 0, 0);
```

### 3. 변환 히스토리

최근 변환 기록 저장

```javascript
const history = JSON.parse(localStorage.getItem('convert-history') || '[]');
history.unshift({
  date: new Date().toISOString(),
  from: inputFormat,
  to: outputFormat,
  originalSize: file.size,
  convertedSize: blob.size
});
localStorage.setItem('convert-history', JSON.stringify(history.slice(0, 10)));
```

## 개발 로그

### 2025-10-25
- 프로젝트 폴더 생성
- README.md 작성
- **경쟁사 분석 완료:**
  - CloudConvert, Convertio, Online-Convert, FreeConvert, iLoveIMG 조사
  - 대부분 횟수/용량 제한, 회원가입 필요
  - 차별화: 완전 무료, 브라우저 기반, 최신 포맷 지원
- **라이브러리 조사 완료:**
  - Canvas API + browser-image-compression 조합 (추천)
  - WebP/AVIF 브라우저 호환성 체크 필수
  - Best practices: 투명도 처리, 품질 최적화, 브라우저 폴백
- **실제 이슈 파악:**
  - PNG 투명 배경 손실, WebP/AVIF 호환성
  - GIF 애니메이션 손실, 메모리 부족
  - JPEG 재압축 품질 저하
- **UI/UX 패턴:**
  - 포맷 선택 (JPEG/PNG/WebP/AVIF)
  - 품질 슬라이더, 고급 옵션 (투명도, 최적화)
  - 전/후 용량 비교, ZIP 다운로드
- **통합 전략:**
  - 압축/리사이즈/업스케일과 탭으로 연결
  - 워크플로우: 압축 → 변환, 리사이즈 → 변환

## 참고 자료

- [Canvas toBlob - MDN](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toBlob)
- [WebP Browser Support - Can I Use](https://caniuse.com/webp)
- [AVIF Browser Support - Can I Use](https://caniuse.com/avif)
- [browser-image-compression GitHub](https://github.com/Donaldcwl/browser-image-compression)
- [Image Format Comparison 2025](https://cloudinary.com/guides/image-formats/webp-vs-jpeg-vs-avif)
