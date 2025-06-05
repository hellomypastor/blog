title: ������ȡ�����ѡ�AI ÿ�����š�
date: 2025-06-05 12:30:46
categories: Personal
tags: [GithubActions,Vercel,AI,RSS]
---
> <font style="color:rgb(31, 35, 40);">���� RSS ��ַ��ÿ�춨ʱ�ɼ�һ��ǰһ������ݣ�ͨ�� GPT ���з����ܽᣬ�γ�һ��ÿ�ձ��棬</font>���� Github Actioins & Vercel <font style="color:rgb(31, 35, 40);">һ��������������ÿ����������,֧�� OpenAI��Gemini Pro��Qwen3 ģ�͡�</font>
>
> �ҵ�ÿ��������ҳ��[https://ai.anchen.me/](https://ai.anchen.me/)
>

<!--more-->

<h3 id="GW1td">Ե��</h3>
AI ����һ���籩��������� ChatGPT���� Copilot RAG���� Workflow Agent���� Cursor���ٵ� Devin����չ���ٶ�֮�죬�������죬��Ҫʱ�̹�עҵ����¼������·��򣬿�����ÿ��������ȥ�Ѽ���Ϣ�Ѿ��������������ˣ��������룬�Ƿ�������Զ����ķ�ʽȥ�Ѽ���Ϣ��ͨ�� AI ȥ�����ܽᣬ�γ�һ�ݱ��档



�����£������и���Դ���߿��������ҵ�����[https://github.com/zhangferry/AIDailyNews](https://github.com/zhangferry/AIDailyNews)

<h3 id="UL7ED">ʵ�ֹ���</h3>
<h4 id="AqH0p">Fork ��Ŀ</h4>
![](������ȡ�����ѡ�AI ÿ�����š�/fork.png)

�ҵĲֿ��ַ��[https://github.com/hellomypastor/daily-news](https://github.com/hellomypastor/daily-news)

<h4 id="XnX8O"><font style="color:rgb(25, 27, 31);">���Ի�����</font></h4>
<font style="color:rgb(25, 27, 31);">�ڲֿ�Ŀ¼���ҵ� workflow/resources/rss.json���ҵ��������ݲ��޸ĳ����Լ���Ҫ�ģ�</font>

```json
{
  "configuration": {
    "rsshub_domain": "https://rsshub.zhangferry.com/"
  },
  "categories": [
    {
      "category": "\uD83E\uDD16 AI News",
      "items": [
        {
          "title": "oneminute_daily_ai_news RSS",
          "url": "https://www.reddit.com/user/Excellent-Target-847/.rss",
          "type": "link",
          "output_count": 1
        },
        {
          "title": "Hacker News - AI",
          "url": "https://hnrss.org/newest?q=AI",
          "type": "link",
          "output_count": 3
        },
        {
          "title": "36Kr",
          "url": "https://rss.aishort.top/?type=36kr",
          "output_count": 1
        },
        {
          "title": "AIShort",
          "url": "https://rss.aishort.top/?type=wasi",
          "type": "link",
          "output_count": 0
        }
      ]
    }
  ]
}
```

<h4 id="H6sLM">ģ��֧��</h4>
�ֿ���ֻ֧���� OpenAI �� Gemini ģ�ͣ������������޸ģ�֧���� Qwen3 ģ�ͣ�����ɲ鿴 /workflow/gpt/requests.py ����

```python
def request_siliconflow(provider: AIProvider, prompt, content):
    """
    https://docs.siliconflow.cn/cn/api-reference/chat-completions/chat-completions
    """
    client = OpenAI(api_key=provider.api_key,
                    base_url=provider.base_url)

    chat_completion = client.chat.completions.create(messages=[
        {
            "role": "system",
            "content": prompt
        },
        {
            "role": "user",
            "content": content
        }
    ], model=provider.model)
    return chat_completion.choices[0].message.content
```

��Ϊ Qwen3 �� API ������ OpenAI �� sdk������ʵ�������Ƚϼ򵥡�

> ��ʹ�õ��� ������� ƽ̨��[https://cloud.siliconflow.cn/models](https://cloud.siliconflow.cn/models)��ʹ�õ��� Qwen3 8B �����ģ��
>
> ![](������ȡ�����ѡ�AI ÿ�����š�/siliconflow-1.png)
>
> ![](������ȡ�����ѡ�AI ÿ�����š�/siliconflow-2.png)
>

<h4 id="PI27G">ͬ������</h4>
<font style="color:rgb(25, 27, 31);">�ڲֿ�Ŀ¼���ҵ� .github/workflows/main.yml���޸�ͬ�����ã�</font>

```yaml

name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: 0 15 * * *  # every 23:59 UTC +8

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  create-daily-news:
    runs-on: ubuntu-latest

    env:
      GIT_NAME: ${{ secrets.GIT_NAME }}
      GIT_EMAIL: ${{ secrets.GIT_EMAIL }}
      AI_PROVIDER: ${{ secrets.AI_PROVIDER }}
      GPT_MODEL_NAME: ${{ secrets.GPT_MODEL_NAME }}
      GPT_BASE_URL: ${{ secrets.GPT_BASE_URL }}
      GPT_API_KEY: ${{ secrets.GPT_API_KEY }}

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Before Execute
        # You may pin to the exact commit or the version.
        run: |
          echo $GPT_API_KEY
          echo $AI_PROVIDER
          ls -l

      - name: Set up Python
        uses: actions/setup-python@v2
        with:
          python-version: '3.x'

      - name: Install requirements
        run: |
          python3 -m pip install --upgrade pip
          pip3 install -r ./requirements.txt

      - name: Create Daily News
        # You may pin to the exact commit or the version.
        run: python3 ./main.py

      - name: Get branch name
        run: echo "BRANCH_NAME=${GITHUB_REF#refs/heads/}" >> $GITHUB_ENV

      - name: Commit
        run: |
          git config --local user.name $GIT_NAME
          git config --local user.email $GIT_EMAIL
          git add .
          git commit -m "Github action update at `date '+%Y-%m-%d %H:%M:%S'`."

      - name: Push
        uses: ad-m/github-push-action@master
        if: ${{ env.BRANCH_NAME == 'main' }} # only for main
        with:
          github_token: ${{ secrets.ACCESS_TOKEN }}
          branch: main

```

<h4 id="LQuSS">���� & ��ʱͬ��</h4>
ʹ�� Vercel �����Զ�������ַ�еĸ���·����Ϣ����

[https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fhellomypastor%2FAIDailyNews&teamSlug=hellomypastors-projects](https://vercel.com/new/clone?repository-url=https%3A%2F%2Fgithub.com%2Fhellomypastor%2FAIDailyNews&teamSlug=hellomypastors-projects)

![](������ȡ�����ѡ�AI ÿ�����š�/vercel-new.png)

�� Vercel ������ Deploy Hooks��

![](������ȡ�����ѡ�AI ÿ�����š�/vercel-hook-1.png)

�ڲֿ������� Webhooks��

![](������ȡ�����ѡ�AI ÿ�����š�/vercel-hook-2.png)

<font style="color:rgb(25, 27, 31);">Ϊ GitHub Actions ��Ӵ����ύȨ�ޣ����ʲֿ�� Settings > Actions > Generalҳ�棬�ҵ� Workflow permissions ���������ѡ������Ϊ Read and write permissions��֧�� CI �����ݸ��º��ύ���ֿ���</font>

![](������ȡ�����ѡ�AI ÿ�����š�/workflow-permission.png)

<font style="color:rgb(25, 27, 31);">���ʲֿ�� Settings > Security > Secrets and variables > Actions ҳ�棬����˺ż���Կ��Ϣ</font>

![](������ȡ�����ѡ�AI ÿ�����š�/action.png)

<h4 id="OtnTy">�Զ�������</h4>
Vercel �ṩ���������������Ƚϳ������׼�ס

![](������ȡ�����ѡ�AI ÿ�����š�/domain-1.png)

�����Զ���������

![](������ȡ�����ѡ�AI ÿ�����š�/domain-2.png)

<h4 id="DvPwg">Ч��</h4>
Vercel ����ҳ�棺

![](������ȡ�����ѡ�AI ÿ�����š�/vercel-deploy.png)

���ɵ���������ҳ�棺

![](������ȡ�����ѡ�AI ÿ�����š�/demo.png)

<h3 id="e5gcs">ԭ��</h3>
![](������ȡ�����ѡ�AI ÿ�����š�/principle.png)

> 1. <font style="color:rgb(31, 35, 40);">GitHub Actions �Զ��� RSS �ɼ�����</font>
> 2. <font style="color:rgb(25, 27, 31);">�ɼ�������� ����� AI��AI ���з����ܽ������ MarkDown �ĵ���push ���ֿ���</font>
> 3. <font style="color:rgb(31, 35, 40);">ͨ�� Vercel �Զ�����</font>
> 4. <font style="color:rgb(31, 35, 40);">֧�� OpenAI��Gemini��Qwen3 ��ģ��</font>
>

<h3 id="YL2kJ">д�����</h3>
��������̲���ʱ 1 Сʱ�����˲����ʱ�������˼�����֮�⣬��������˳��������Ȥ��ͬѧ�������ԡ�

�� AI ȥ�����Ƿ����ܽ� AI ��ص����ţ���������ҿ������Ǻ���Ȥ�ܿ�ġ�AI ʱ���������������л��ᣬ��Ҫ�£������ȥ����ֵ�����ʤ����

<h3 id="nwIXL">�ο�</h3>
��1��[https://github.com/zhangferry/AIDailyNews](https://github.com/zhangferry/AIDailyNews)

