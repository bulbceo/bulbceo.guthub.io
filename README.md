# bulbceo.guthub.io
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>한글 단어 플래시 카드</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'Noto Sans KR', 'sans-serif'],
                    },
                }
            }
        }
    </script>
    <style>
        /* 16:9 비율을 유지하는 컨테이너 스타일 */
        .aspect-ratio-16x9 {
            width: 100%;
            padding-bottom: 56.25%; /* 9 / 16 = 0.5625 */
            position: relative;
        }
        .content-area {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
        }
    </style>
</head>
<body class="bg-gray-100 min-h-screen flex items-center justify-center p-4 font-sans">

    <div class="w-full max-w-4xl shadow-2xl rounded-xl bg-white overflow-hidden">
        <!-- 앱 제목 -->
        <div class="p-4 bg-indigo-600">
            <h1 class="text-xl sm:text-2xl text-white font-bold text-center">
                받침 없는 두 글자 단어 학습 (온전한 형태)
            </h1>
        </div>

        <!-- 16:9 비율 컨테이너 -->
        <div class="aspect-ratio-16x9">
            <div class="content-area">
                
                <!-- 단어 표시 영역: flex를 사용하여 글자 사이 간격 조정 및 중앙 정렬 -->
                <div id="wordDisplay" class="flex justify-center text-8xl sm:text-[10rem] md:text-[15rem] font-extrabold transition-opacity duration-300 transform scale-100 select-none leading-none text-gray-800">
                    <!-- 온전한 단어가 여기에 표시됩니다 (예: 바다) -->
                </div>
                
                <!-- 왼쪽 버튼 -->
                <button id="prevButton"
                    class="absolute left-0 top-0 h-full w-1/12 md:w-1/16 bg-black bg-opacity-10 hover:bg-opacity-20 transition duration-300 text-white text-4xl sm:text-6xl rounded-l-xl opacity-0 cursor-default disabled:opacity-0 disabled:cursor-default"
                    disabled>
                    <!-- Font Awesome icon (Arrow Left) -->
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" class="h-8 w-8 mx-auto fill-current">
                        <path d="M9.4 233.4c-12.5 12.5-12.5 32.8 0 45.3l160 160c12.5 12.5 32.8 12.5 45.3 0s12.5-32.8 0-45.3L109.2 288 416 288c17.7 0 32-14.3 32-32s-14.3-32-32-32l-306.7 0L214.6 118.6c12.5-12.5 12.5-32.8 0-45.3s-32.8-12.5-45.3 0l-160 160z"/>
                    </svg>
                </button>

                <!-- 오른쪽 버튼 (강조된 색상으로 변경) -->
                <button id="nextButton"
                    class="absolute right-0 top-0 h-full w-1/12 md:w-1/16 bg-amber-500 hover:bg-amber-600 transition duration-300 text-white text-4xl sm:text-6xl rounded-r-xl">
                    <!-- Font Awesome icon (Arrow Right) -->
                    <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 448 512" class="h-8 w-8 mx-auto fill-current">
                        <path d="M438.6 278.6c12.5-12.5 12.5-32.8 0-45.3l-160-160c-12.5-12.5-32.8-12.5-45.3 0s-12.5 32.8 0 45.3L338.8 224 32 224c-17.7 0-32 14.3-32 32s14.3 32 32 32l306.7 0L233.4 394.6c-12.5 12.5-12.5 32.8 0 45.3s32.8 12.5 45.3 0l160-160z"/>
                    </svg>
                </button>

            </div>
        </div>
        <!-- 현재 인덱스 및 설명 -->
        <div class="p-3 text-center bg-gray-50 border-t">
            <p id="wordCount" class="text-sm font-medium text-gray-500 mt-1">
                새로운 단어 (New Word)
            </p>
        </div>
    </div>

    <script>
        // 받침(종성)이 없는 두 글자 단어 목록
        const NO_BATCHIM_WORDS = [
            '바다', '나비', '포도', '나무', '소라', '두부', '기차', '모자', '아기', '허리',
            '가구', '다리', '사자', '타조', '하마', '고기', '요가', '비누', '구두', '제도'
        ];

        // 상태 관리
        let wordHistory = [];
        let currentIndex = -1;

        const wordDisplay = document.getElementById('wordDisplay');
        const prevButton = document.getElementById('prevButton');
        const nextButton = document.getElementById('nextButton');
        const wordCountDisplay = document.getElementById('wordCount');
        
        /**
         * 단어를 온전한 형태로 화면에 표시하고 단어 사이 간격을 넓힙니다.
         * @param {string} word - 표시할 두 글자 단어
         * @param {string} direction - 애니메이션 방향 ('next' 또는 'prev')
         */
        function displayWord(word, direction = 'next') {
            if (!word || word.length !== 2) return; 

            wordDisplay.classList.remove('scale-100');
            wordDisplay.classList.add('scale-90', 'opacity-0');
            
            setTimeout(() => {
                // 단어와 단어 사이를 넓히기 위해 각 글자를 <span>으로 감쌉니다.
                const syllable1 = word[0];
                const syllable2 = word[1];
                
                // mr-24 (6rem, 96px)을 사용하여 단어 사이를 넓게 벌립니다.
                wordDisplay.innerHTML = `
                    <span class="mr-24">${syllable1}</span>
                    <span>${syllable2}</span>
                `;

                // 애니메이션 복원
                wordDisplay.classList.remove('scale-90', 'opacity-0');
                wordDisplay.classList.add('scale-100', 'opacity-100');
                
                // 버튼 상태 업데이트
                updateButtons();
                updateWordCount();

            }, 200); // 0.2초 후 내용 변경
        }

        /**
         * 다음 단어를 로드하거나 새로운 단어를 생성합니다.
         */
        function loadNextWord() {
            let nextWord;
            
            if (currentIndex < wordHistory.length - 1) {
                // 기록에 남아있는 단어가 있다면 다음 단어를 보여줍니다.
                currentIndex++;
                nextWord = wordHistory[currentIndex];
            } else {
                // 새로운 단어를 랜덤으로 선택합니다.
                const newWordIndex = Math.floor(Math.random() * NO_BATCHIM_WORDS.length);
                nextWord = NO_BATCHIM_WORDS[newWordIndex];
                
                // 단어 기록에 추가하고 인덱스 업데이트
                wordHistory.push(nextWord);
                currentIndex = wordHistory.length - 1;
            }

            displayWord(nextWord, 'next');
        }

        /**
         * 이전 단어를 로드합니다.
         */
        function loadPrevWord() {
            if (currentIndex > 0) {
                currentIndex--;
                const prevWord = wordHistory[currentIndex];
                displayWord(prevWord, 'prev');
            }
        }

        /**
         * 좌우 버튼의 활성화/비활성화 상태를 업데이트합니다.
         */
        function updateButtons() {
            // 이전 버튼 활성화: currentIndex가 0보다 클 때
            if (currentIndex > 0) {
                prevButton.disabled = false;
                prevButton.classList.remove('opacity-0', 'cursor-default');
                prevButton.onclick = loadPrevWord; // 이벤트 핸들러 재설정
            } else {
                prevButton.disabled = true;
                prevButton.classList.add('opacity-0', 'cursor-default');
                prevButton.onclick = null; // 이벤트 제거
            }

            // 다음 버튼은 항상 활성화되어 새 단어를 로드하거나 기록을 순회할 수 있게 함
            nextButton.disabled = false;
        }

        /**
         * 단어 카운터 텍스트를 업데이트합니다.
         */
        function updateWordCount() {
            if (currentIndex === wordHistory.length - 1) {
                wordCountDisplay.textContent = '새로운 단어를 로드하세요 (Next Word)';
            } else {
                wordCountDisplay.textContent = `단어 ${currentIndex + 1} / ${wordHistory.length} (Viewing History)`;
            }
        }

        // 초기 로드: 첫 번째 단어를 표시
        window.onload = function() {
            // 스크립트 로드 완료 후 이벤트 핸들러를 붙입니다.
            prevButton.onclick = loadPrevWord;
            nextButton.onclick = loadNextWord;

            loadNextWord(); // 첫 단어를 로드하면서 history에 추가
        };
    </script>
</body>
</html>
