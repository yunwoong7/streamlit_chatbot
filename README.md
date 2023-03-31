

<h2 align="center">
Streamlit Chatbot
</h2>

<div align="center">
  <img src="https://img.shields.io/badge/python-v3.9.16-blue.svg"/>
  <img src="https://img.shields.io/badge/streamlit-v1.20.0-blue.svg"/>
  <img src="https://img.shields.io/badge/streamlit_chat-v0.0.2.2-blue.svg"/>
</div>

**streamlit-chat**은 Streamlit을 이용하여 실시간 대화형 Web 애플리케이션을 쉽게 만들 수 있는 패키지입니다. 만드는 방법은 매우 간단하며 [예제](https://share.streamlit.io/ai-yash/st-chat/main/examples/chatbot.py) 확인도 가능합니다.



<div align="center">
<img src="https://blog.kakaocdn.net/dn/cACrfC/btr65p6tNGQ/kPoXKMh4LAmUtsuhGXOHOk/img.gif" width="70%">
</div>

streamlit-chat으로 Hugging Face에서 제공하는 Facebook AI BlenderBot과 OpenAI의 GPT-3 모델로 챗봇 프로그램을 만들어 보겠습니다.

------

### 1. 설치 (Installation)

streamlit과 streamlit-chat 패키지 설치는 pip 명령어를 이용하여 설치할 수 있습니다.

```bash
pip install streamlit
pip install streamlit-chat
```

streamlit 설치와 관련된 내용은 이전 글을 참고하시기 바랍니다.

 [Streamlit 시작하기 (설치방법)](https://yunwoong.tistory.com/226)

### 2. BlenderBot 챗봇

BlenderBot은 Hugging Face의 Transformers 라이브러리를 이용하여 쉽게 사용할 수 있습니다. 먼저 Hugging Face Inference API Key 발급을 진행합니다. 아래 글을 참고하세요.

  [Hugging Face Inference API Key 발급](https://yunwoong.tistory.com/225)

Python 파일 blenderbot_app.py 을 생성하고 아래와 같이 작성합니다. API_TOKEN은 자신의 Hugging Face Inference API Key 를 입력합니다. (예: hf_xxxxxxxxxxxxxxxxxxxxx)

```python
import streamlit as st
from streamlit_chat import message
import requests
 
API_URL = "https://api-inference.huggingface.co/models/facebook/blenderbot-400M-distill"
API_TOKEN = "YOUR API TOKEN HERE"
headers = {"Authorization": f"Bearer {API_TOKEN}"}
 
st.header("🤖Yunwoong's BlenderBot (Demo)")
st.markdown("[Be Original](https://yunwoong.tistory.com/)")
 
if 'generated' not in st.session_state:
    st.session_state['generated'] = []
 
if 'past' not in st.session_state:
    st.session_state['past'] = []
 
def query(payload):
	response = requests.post(API_URL, headers=headers, json=payload)
	return response.json()
 
 
with st.form('form', clear_on_submit=True):
    user_input = st.text_input('You: ', '', key='input')
    submitted = st.form_submit_button('Send')
 
if submitted and user_input:
    output = query({
        "inputs": {
            "past_user_inputs": st.session_state.past,
            "generated_responses": st.session_state.generated,
            "text": user_input,
        },
        "parameters": {"repetition_penalty": 1.33},
    })
 
    st.session_state.past.append(user_input)
    st.session_state.generated.append(output["generated_text"])
 
if st.session_state['generated']:
    for i in range(len(st.session_state['generated'])-1, -1, -1):
        message(st.session_state['past'][i], is_user=True, key=str(i) + '_user')
        message(st.session_state["generated"][i], key=str(i))
```

터미널에서 아래와 같이 입력하면 Streamlit 앱을 실행합니다.

```
streamlit run blenderbot_app.py
```

<div align="center">
<img src="https://blog.kakaocdn.net/dn/nwBzi/btr7cISnZwI/JK6JWPNCeEUNtBGRShOGFK/img.gif" width="70%">
</div>

### 3. GPT-3 챗봇

다음으로 OpenAI API를 이용한 GPT-3 챗봇을 만들도록 하겠습니다.

먼저 OpenAI API를 사용하기 위해 API 키 발급이 필요합니다. 먼저 [OpenAI API 사이트](https://platform.openai.com/)로 이동합니다. OpenAI 계정이 필요하며 계정이 없다면 계정 생성이 필요합니다. 간단히 Google이나 Microsoft 계정을 연동 할 수 있습니다. 이미 계정이 있다면 로그인 후 진행하시면 됩니다.

로그인이 되었다면 우측 상단 Personal -> [ View API Keys ] 를 클릭합니다.

![img](https://blog.kakaocdn.net/dn/xKSqg/btr62GPoKvC/OF7uLj6YZhmv1VkVyDOJN0/img.png)

[ + Create new secret key ] 를 클릭하여 API Key를 생성합니다. API key generated 창이 활성화되면 Key 를 반드시 복사하여 두시기 바랍니다. 창을 닫으면 다시 확인할 수 없습니다. (만약 복사하지 못했다면 다시 Create new secret key 버튼을 눌러 생성하면 되니 걱정하지 않으셔도 됩니다.)

Python 파일 chatgpt_app.py 을 생성하고 아래와 같이 작성합니다. openai.api_key는 자신의 OpenAI API Key를 입력합니다. (예: sk-xxxxxxxxxxxxxxxxxxxxx)

```
import openai
import streamlit as st
from streamlit_chat import message
 
openai.api_key = 'YOUR API KEY HERE'
 
def generate_response(prompt):
    completions = openai.Completion.create (
        engine="text-davinci-003",
        prompt=prompt,
        max_tokens=1024,
        stop=None,
        temperature=0,
        top_p=1,
    )
 
    message = completions["choices"][0]["text"].replace("\n", "")
    return message
 
 
st.header("🤖Yunwoong's ChatGPT-3 (Demo)")
st.markdown("[Be Original](https://yunwoong.tistory.com/)")
 
if 'generated' not in st.session_state:
    st.session_state['generated'] = []
 
if 'past' not in st.session_state:
    st.session_state['past'] = []
 
with st.form('form', clear_on_submit=True):
    user_input = st.text_input('You: ', '', key='input')
    submitted = st.form_submit_button('Send')
 
if submitted and user_input:
    output = generate_response(user_input)
    st.session_state.past.append(user_input)
    st.session_state.generated.append(output)
 
if st.session_state['generated']:
    for i in range(len(st.session_state['generated'])-1, -1, -1):
        message(st.session_state['past'][i], is_user=True, key=str(i) + '_user')
        message(st.session_state["generated"][i], key=str(i))
```

터미널에서 아래와 같이 입력하면 Streamlit 앱을 실행합니다.

```
streamlit run chatgpt_app.py
```

<div align="center">
<img src="https://blog.kakaocdn.net/dn/bceZgF/btr7cdZnqLq/FpszCkE2k72fsQrlS8FSrk/img.gif" width="70%">
</div>

------

매우 간단하게 Web 애플리케이션을 만들어 시뮬레이션이 가능합니다. 만일 개발자가 아닌 데이터 과학자나 AI 모델러인 경우 시뮬레이터를 구축하려면 시간과 노력이 많이 들 수 있지만 Streamlit을 이용한다면 이 과정을 단순화하고 시간을 절약할 수 있을 것 같습니다.
