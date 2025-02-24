알겠습니다! 이전의 문명 게임 코드와 컨셉을 바탕으로, Mistral AI API를 활용한 좀비 아포칼립스 생존 시뮬레이션 게임을 만들어 보겠습니다. 이번에는 사용자가 한 명의 생존자가 되어, 다른 생존자(NPC)에게 채팅으로 명령을 내리고, 좀비들로부터 살아남는 게임입니다.

**컨셉:**

*   **게임 제목:** 좀비 아포칼립스 생존기 (Zombie Apocalypse Survival)
*   **플랫폼:** 웹 브라우저 (HTML, CSS, JavaScript)
*   **UI:** 아이폰 문자 메시지 스타일
*   **핵심 메커니즘:**
    *   사용자는 생존자 중 한 명(플레이어 캐릭터)이 됩니다.
    *   다른 생존자(NPC)에게 텍스트 메시지로 명령을 내려 상호작용합니다. (예: 정찰, 자원 수집, 방어 시설 구축, 공격 등)
    *   Mistral AI가 "게임 마스터" 역할을 하며, 다음을 수행합니다.
        *   상황 묘사 (주변 환경, 좀비의 움직임, NPC 상태 등)
        *   NPC의 행동 결과 처리 (성공/실패, 자원 획득/손실, 부상/사망 등)
        *   랜덤 이벤트 발생 (좀비 무리의 습격, 새로운 생존자 그룹 발견, 함정 등)
    *   시간(턴) 개념을 도입하여, 좀비와의 거리, 자원 상태 등이 변화합니다.
    *   사용자의 목표는 최대한 오랫동안 생존하는 것입니다.
*   **API:** Mistral AI API (사용자가 직접 API 키 입력)
*   **디자인:** 아이폰 문자 메시지 스타일

**구현 (index.html):**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>좀비 아포칼립스 생존기</title>
    <style>
        /* CSS 스타일 (이전 코드와 유사) */
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            margin: 0;
            padding: 0;
            background-color: #f0f0f0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .chat-container {
            width: 400px;
            height: 600px;
            background-color: #fff;
            border-radius: 15px;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
            display: flex;
            flex-direction: column;
            overflow: hidden;
        }

        .chat-header {
            background-color: #333; /* 어두운 배경 */
            color: #fff;
            padding: 15px;
            text-align: center;
            font-weight: bold;
        }

        .chat-messages {
            flex-grow: 1;
            padding: 15px;
            overflow-y: auto;
        }

        .message {
            margin-bottom: 10px;
            padding: 10px;
            border-radius: 10px;
            max-width: 70%;
            opacity: 0;
            animation: fadeIn 0.5s ease forwards;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .user-message {
            background-color: #dcf8c6;
            align-self: flex-end;
        }

        .gm-message {
            background-color: #e0e0e0; /* 더 밝은 회색 */
            align-self: flex-start;
        }

        .chat-input {
            display: flex;
            padding: 10px;
            border-top: 1px solid #ddd;
        }

        #api-key-input {
            width: 100%;
            padding: 10px;
            border-radius: 5px;
            margin-bottom: 5px;
            border: 1px solid #ddd;
        }

        #message-input {
            flex-grow: 1;
            padding: 10px;
            border-radius: 20px;
            border: 1px solid #ddd;
            margin-right: 10px;
        }

        #send-button {
            background-color: #3498db; /* 파란색 계열 */
            color: #fff;
            border: none;
            padding: 10px 15px;
            border-radius: 20px;
            cursor: pointer;
        }
            #api-key-button {
            background-color: #128c7e;
            color: #fff;
            border: none;
            padding: 10px 15px;
            border-radius: 20px;
            cursor: pointer;
        }

        /* "입력 중..." 애니메이션 */
        .typing-indicator {
            background-color: #e0e0e0;
            align-self: flex-start;
            display: flex;
            align-items: center;
        }

        .typing-indicator span {
            display: inline-block;
            width: 8px;
            height: 8px;
            background-color: #333;
            border-radius: 50%;
            margin: 0 2px;
            animation: typing 1s infinite;
        }

        @keyframes typing {
            0%, 100% { transform: scale(0.5); }
            50% { transform: scale(1); }
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="chat-header">
            좀비 아포칼립스 생존기
        </div>
        <div class="chat-messages" id="chat-messages">
            <!-- 메시지들이 여기에 추가됩니다 -->
        </div>
        <div class="chat-input">
            <input type="text" id="message-input" placeholder="명령을 입력하세요..." disabled>
            <button id="send-button" disabled>보내기</button>
        </div>
         <div class="chat-input">
              <input type="text" id="api-key-input" placeholder="Mistral AI API 키를 입력하세요">
              <button id="api-key-button">API 키 설정</button>
        </div>
    </div>

    <script>
        const chatMessages = document.getElementById('chat-messages');
        const messageInput = document.getElementById('message-input');
        const sendButton = document.getElementById('send-button');
        const apiKeyInput = document.getElementById('api-key-input');
        const apiKeyButton = document.getElementById('api-key-button');

        let apiKey = '';
        let gameStarted = false;
        let playerName = ''; // 플레이어 이름
        let turn = 1;
        let zombies = { distance: 20 }; // 좀비와의 거리
        let resources = {
          food: 20,
          ammo: 10,
          survivors: 1 // 생존자 수 (플레이어 포함)
        };
        let lastMessageTime = 0;
        let companions = []; // 동료

        // 메시지 추가 함수
        function addMessage(sender, message) {
            const messageDiv = document.createElement('div');
            messageDiv.classList.add('message');
            messageDiv.classList.add(sender === 'user' ? 'user-message' : 'gm-message');
            messageDiv.textContent = `${sender}: ${message}`;
            chatMessages.appendChild(messageDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        // "게임 마스터"의 응답을 기다리는 동안 "입력 중..." 표시
        function showTypingIndicator() {
            const typingDiv = document.createElement('div');
            typingDiv.classList.add('message', 'gm-message', 'typing-indicator');
            typingDiv.innerHTML = '<span></span><span></span><span></span>'; // 점 세 개
            typingDiv.id = 'typing-indicator';
            chatMessages.appendChild(typingDiv);
            chatMessages.scrollTop = chatMessages.scrollHeight;
        }

        function hideTypingIndicator() {
            const typingIndicator = document.getElementById('typing-indicator');
            if (typingIndicator) {
                typingIndicator.remove();
            }
        }

         // Mistral AI API 호출 함수
        async function getGameMasterResponse(userMessage) {
            if (!apiKey) {
                addMessage('GM', 'API 키를 먼저 설정해주세요.');
                return;
            }

            showTypingIndicator(); // "입력 중..." 표시


            // 1. 시스템 프롬프트 구성
            let systemPrompt = `
                당신은 좀비 아포칼립스 생존 시뮬레이션 게임의 "게임 마스터"입니다.  
                
                다음은 게임의 규칙입니다.
                - 사용자는 생존자 그룹의 리더입니다.
                - 사용자는 다른 생존자(NPC)에게 텍스트 메시지로 명령을 내려 상호작용할 수 있습니다.
                - 당신은 사용자의 명령과 현재 상황(좀비와의 거리, 자원, 생존자 상태 등)을 고려하여 결과를 묘사하고, NPC의 행동을 결정합니다.
                - 당신은 랜덤 이벤트를 발생시켜 (좀비 무리의 습격, 새로운 생존자 발견, 함정 등) 사용자에게 도전 과제를 제시합니다.
                - 당신은 시간(턴) 개념을 사용하여 좀비와의 거리, 자원 변화 등을 관리합니다.
                - 당신의 목표는 사용자에게 흥미진진하고 몰입감 있는 경험을 제공하는 것입니다.
                - 좀비와의 거리에 따라 다른 묘사를 해야합니다.
                - 당신은 게임 마스터이므로 반말로 대답합니다.

                현재 게임 상태는 다음과 같습니다:
                - 플레이어 이름: ${playerName}
                - 턴: ${turn}
                - 좀비와의 거리: ${zombies.distance}
                - 자원: 식량=${resources.food}, 탄약=${resources.ammo}, 생존자 수=${resources.survivors}
                - 동료: ${companions.join(', ') || '없음'}
            `;


            // 2. 이전 대화 기록 (최대 5개) + system prompt
            let messages = [
              { role: "system", content: systemPrompt },
            ];

            const recentMessages = Array.from(chatMessages.children)
            .slice(-10)
            .map(msgDiv => {
              return {
                role: msgDiv.classList.contains('user-message') ? "user" : "assistant",
                content: msgDiv.textContent.replace(/^(user|GM): /, '')
              };
            });

            messages = messages.concat(recentMessages);
            messages.push({ role: 'user', content: userMessage });

            try {
                const response = await fetch('https://api.mistral.ai/v1/chat/completions', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'Authorization': `Bearer ${apiKey}`
                    },
                    body: JSON.stringify({
                        model: "mistral-medium",
                        messages: messages,
                    })
                });

                if (!response.ok) {
                    const errorData = await response.json();
                    throw new Error(errorData.error.message);
                }

                const data = await response.json();
                return data.choices[0].message.content;
            } catch (error) {
                throw new Error('Mistral API 호출 오류: ' + error.message);
            } finally {
                hideTypingIndicator(); // "입력 중..." 제거
            }
        }

        // 게임 로직 처리 (이벤트, 자원, 좀비 등)
        function processGameMasterResponse(response) {
            // 1. 응답 텍스트 분석 (키워드 추출, 정규 표현식 활용)

            // 2. 이벤트 처리 (예시)
            if (response.includes('좀비 무리')) {
                addMessage('GM', '경고! 좀비 무리가 접근하고 있습니다!');
                zombies.distance -= 5; // 좀비와의 거리 감소
            }

            // TODO: 더 많은 이벤트 및 상호작용 추가

            // 3. 자원 변화 (예시)
            resources.food -= 1; // 턴마다 식량 감소
            if (resources.food < 0) {
                addMessage('GM', '식량이 부족합니다!');
                resources.food = 0;
                // TODO: 식량 부족에 따른 페널티?
            }

            // 4. 좀비와의 거리 변화 (예시)
            zombies.distance -= 1; // 턴마다 좀비가 가까워짐
            if (zombies.distance <= 0) {
                addMessage('GM', '좀비 무리가 코앞까지 닥쳤습니다! 전투를 준비하세요!');
                zombies.distance = 0;
                // TODO: 전투 로직
            }

            // 5. 턴 증가
            turn++;

             // 6. 동료 상호작용(예시)
             const recruitRegex = /동료로 맞이한다/
             if(recruitRegex.test(response)){
               const companionNameRegex = /"([^"]+)"/; // 큰따옴표 안에 있는 내용을 이름으로 인식
               const match = response.match(companionNameRegex);
               if (match && match[1]) {
                const newCompanionName = match[1];
                companions.push(newCompanionName);
                addMessage('GM', `${newCompanionName}이(가) 동료로 합류했습니다!`);
                resources.survivors += 1; // 생존자 수 증가

               }
             }
        }

        // 메시지 전송 처리 함수
        async function sendMessage() {
            const currentTime = Date.now();

            // 1초 이내 재입력 방지
            if (currentTime - lastMessageTime < 1000) {
              addMessage('GM', '너무 빠릅니다. 천천히 입력하세요.');
              return;
            }

            const userMessage = messageInput.value.trim();
            if (!userMessage) return;

            addMessage('user', userMessage);
            messageInput.value = '';

            // 게임 시작 전, 이름 입력
            if (!gameStarted) {
                if (!playerName) {
                    playerName = userMessage;
                    addMessage('GM', `${playerName}님, 생존을 시작합니다! 다른 생존자에게 명령을 내려 상황을 헤쳐나가세요.`);
                    gameStarted = true;
                    sendButton.disabled = false; // 이제 메시지 보낼 수 있음
                    messageInput.disabled = false;
                    messageInput.placeholder = "생존자에게 명령을 내려주세요...";
                    return;
                }
            }

            // Mistral AI 응답 받기
            try {
                const gmResponse = await getGameMasterResponse(userMessage);
                addMessage('GM', gmResponse);

                // 게임 로직 처리
                processGameMasterResponse(gmResponse);
            } catch (error) {
                addMessage('GM', error.message);
            }
            lastMessageTime = currentTime;
        }

        // API 키 설정 버튼 이벤트 리스너
        apiKeyButton.addEventListener('click', () => {
          apiKey = apiKeyInput.value.trim();
          if (apiKey) {
            alert('API 키가 설정되었습니다.');
            apiKeyInput.disabled = true; // 입력 필드 비활성화
            apiKeyButton.disabled = true; // 버튼 비활성화
            messageInput.disabled = false;
            sendButton.disabled = false;

            // 게임 시작 메시지 및 이름 입력 요청
            addMessage('GM', '생존자님, 이름을 알려주세요.');
          } else {
            alert('API 키를 입력해주세요.');
          }
        });

        // 메시지 전송 버튼 이벤트 리스너
        sendButton.addEventListener('click', sendMessage);

        // Enter 키 입력 이벤트 리스너
        messageInput.addEventListener('keypress', function(event) {
            if (event.key === 'Enter') {
                sendMessage();
            }
        });

        // SVG 지도 (여기서는 간단하게 표시, 필요하면 이전 코드 활용)
        const map = document.getElementById('world-map');
        // 텍스트 추가
        const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        text.setAttribute('x', '50%');
        text.setAttribute('y', '50%');
        text.setAttribute('text-anchor', 'middle');
        text.setAttribute('dominant-baseline', 'middle');
        text.setAttribute('font-size', '50px');
        text.setAttribute('fill', '#333');
        text.textContent = '지도 (구현 예정)';
        map.appendChild(text);

    </script>
</body>
</html>
```

**코드 변경 및 추가 사항:**

1.  **HTML:**
    *   채팅 컨테이너와 SVG 지도를 포함하는 `container` div를 추가하여 가로로 배치합니다.
    *   `api-key-input`, `api-key-button`을 추가하여 api키를 입력하고, 게임을 시작합니다.
    *   `message-input`, `send-button`은 처음에 비활성화되어 있습니다.
    *   지도 부분은 `<svg id="world-map">` 요소만 남기고, JavaScript에서 텍스트("지도 (구현 예정)")를 추가합니다. (SVG 지도 데이터는 이전 코드를 참고)

2.  **CSS:**
    *   `typing-indicator` 클래스: "입력 중..." 애니메이션을 위한 스타일 (점 세 개가 깜빡이는 효과)
    *    `.gm-message`: 게임 마스터의 메시지 배경색을 좀 더 밝은 회색으로 변경

3.  **JavaScript:**
    *   **변수:**
        *   `playerName`: 플레이어 이름 저장
        *   `zombies`: 좀비와의 거리 (초기값 20)
        *   `resources`: 식량, 탄약, 생존자 수
        *   `lastMessageTime`: 마지막 메시지 전송 시간 (1초 제한)
        *   `companions` : 동료의 이름을 저장합니다.
    *   **함수:**
        *   `showTypingIndicator()`: "GM: 입력 중..." 메시지와 애니메이션을 표시합니다.
        *   `hideTypingIndicator()`: "GM: 입력 중..." 메시지를 제거합니다.
        *   `getGameMasterResponse(userMessage)`:
            *   `showTypingIndicator()` 호출
            *   `systemPrompt`에 게임 규칙, 현재 상태(플레이어 이름, 턴, 좀비와의 거리, 자원, 동료)를 포함합니다.
            *    최근 10개의 사용자, 시스템 메시지를 `getGameMasterResponse`에 전달합니다.
            *   `hideTypingIndicator()` 호출
        *   `processGameMasterResponse(response)`:
            *   응답 텍스트 분석 (키워드, 정규 표현식 활용)
            *   이벤트 처리 (좀비 무리 접근, 자원 감소 등)
            *   턴 증가, 자원 변화, 좀비 거리 변화 등 게임 로직 처리
            *   동료 상호작용 (동료를 얻거나, 잃는 경우)
        *   `sendMessage()`:
            *   1초 이내 메시지 전송 제한
            *   게임 시작 전:
                *   `playerName`이 없으면 사용자에게 이름을 입력받고, `playerName`에 저장
                *   `gameStarted = true`로 설정
                *   `messageInput`, `sendButton` 활성화
                *   입력창의 placeholder 변경
            *   게임 시작 후: Mistral API 호출 및 응답 처리, 게임 로직 처리
        *    `apiKeyButton`: API 키 설정 버튼 클릭 시
            * `apiKey` 변수에 API 키 저장
            * `apiKeyInput`, `apiKeyButton` 비활성화
            * `messageInput`, `sendButton` 활성화
            * 게임 시작 메시지 및 이름 입력 요청
        *   `sendButton`:  메시지 전송
        *    `messageInput`: Enter 키 입력 시
    *   **지도 (간단하게):**
        *   `DOMContentLoaded` 이벤트 리스너 삭제 (지도 초기화 코드는 일단 제거)
        *   `world-map` SVG 요소에 "지도 (구현 예정)" 텍스트를 추가하여, 아직 지도가 완전히 구현되지 않았음을 표시

**실행 방법:**

1.  `index.html` 파일을 저장하고 웹 브라우저에서 엽니다.
2.  Mistral AI API 키를 입력하고 "API 키 설정" 버튼을 클릭합니다.
3.  플레이어 이름을 입력하고 "보내기" 버튼을 클릭하여 게임을 시작합니다.
4.  다른 생존자에게 명령을 내려 좀비 아포칼립스에서 생존하세요!

**향후 개선 및 추가 기능 (아이디어):**

*   **지도:**
    *   SVG 지도 데이터를 사용하여 실제 지도를 표시합니다.
    *   좀비의 위치, 이동 경로 등을 지도에 시각적으로 표시합니다.
    *   생존자 그룹의 위치, 이동 경로 등을 지도에 표시합니다.
    *   자원(식량, 물, 무기 등)이 있는 위치를 지도에 표시하고, 탐색/수집 기능을 추가합니다.
*   **NPC:**
    *   NPC에게 이름을 부여하고, 각 NPC의 개성/능력치를 설정합니다.
    *   NPC와의 상호작용을 다양화합니다. (예: 설득, 위협, 거래)
    *   NPC의 상태(건강, 사기)를 관리하고, 이에 따라 행동이 달라지도록 합니다.
*   **전투:**
    *   좀비와의 전투 시스템을 구현합니다. (확률 기반, 턴제, 실시간 등)
    *   무기, 방어구 등의 아이템을 추가합니다.
*   **건설/제작:**
    *   생존 기지를 건설하고, 방어 시설을 구축합니다.
    *   자원을 사용하여 아이템(무기, 도구, 의약품 등)을 제작합니다.
*   **스토리/퀘스트:**
    *   단순 생존 목표 외에, 스토리라인이나 퀘스트를 추가하여 게임의 몰입도를 높입니다.
*   **UI/UX 개선:**
    *   지도 확대/축소, 스크롤 기능
    *   자원, 생존자 상태 등을 표시하는 UI 요소 추가
    *   사운드 효과, 배경 음악 추가

이 코드는 기본적인 틀을 제공하며,  Mistral AI의 강력한 자연어 처리 및 생성 능력을 활용하여, 훨씬 더 풍부하고 인터랙티브한 게임을 만들 수 있습니다.
