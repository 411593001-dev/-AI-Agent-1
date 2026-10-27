# -AI-Agent-1
用程式做一個簡單的 AI Agent

import gradio as gr
from openai import OpenAI


def run_agent(api_key, user_text, writing_style):
    if not api_key or not api_key.strip():
        return "", "", "請先輸入 OpenAI API Key。"

    if not user_text or not user_text.strip():
        return "", "", "請先輸入你想修改的文字。"

    try:
        client = OpenAI(api_key=api_key.strip())

        # Step 1：產生初稿
        draft_response = client.responses.create(
            model="gpt-4.1-mini",
            input=f"""
你是一位寫作助手。
請依照使用者的需求撰寫一份初稿。

寫作風格：{writing_style}
使用者輸入：{user_text}

請直接輸出初稿，不要先評論。
"""
        )
        draft = draft_response.output_text

        # Step 2：Reflection，自我反思
        reflection_response = client.responses.create(
            model="gpt-4.1-mini",
            input=f"""
你是一位嚴謹的文章編輯。

請檢查以下初稿：
1. 是否符合使用者的需求？
2. 內容是否清楚、完整？
3. 是否有重複、空泛或不自然的表達？
4. 可以如何改善？

使用者原始需求：
{user_text}

初稿：
{draft}

請列出具體的問題及修改方向。
如果沒有明顯問題，也請如實說明。
"""
        )
        reflection = reflection_response.output_text

        # Step 3：依照反思結果修改
        final_response = client.responses.create(
            model="gpt-4.1-mini",
            input=f"""
請根據以下資訊修改文章。

使用者需求：
{user_text}

初稿：
{draft}

反思與修改建議：
{reflection}

請保留原本的核心意思，改善文章品質，
並符合使用者指定的寫作風格：{writing_style}。

只輸出修改後的完整文章。
"""
        )
        final_text = final_response.output_text

        return draft, reflection, final_text

    except Exception as e:
        return "", "", f"執行失敗：{str(e)}"


with gr.Blocks(title="AI 自我反思寫作助手") as demo:
    gr.Markdown("# ✨ AI 自我反思寫作助手")
    gr.Markdown(
        "使用 Reflection 方法，讓 AI 經過初稿生成、"
        "自我反思與修改，提升文章品質。"
    )

    api_key = gr.Textbox(
        label="OpenAI API Key",
        type="password",
        placeholder="請輸入自己的 API Key"
    )

    user_text = gr.Textbox(
        label="輸入你的寫作需求",
        placeholder="例如：幫我寫一篇參加文學講座的心得，約 200 字。",
        lines=5
    )

    writing_style = gr.Dropdown(
        choices=["自然口語", "正式報告", "簡潔有力", "溫暖真誠"],
        value="自然口語",
        label="選擇寫作風格"
    )

    submit_btn = gr.Button("開始反思與修改", variant="primary")

    draft_output = gr.Textbox(
        label="第一階段：AI 初稿",
        lines=6
    )

    reflection_output = gr.Textbox(
        label="第二階段：AI 反思分析",
        lines=6
    )

    final_output = gr.Textbox(
        label="第三階段：最終修改結果",
        lines=8
    )

    submit_btn.click(
        fn=run_agent,
        inputs=[api_key, user_text, writing_style],
        outputs=[draft_output, reflection_output, final_output]
    )

demo.launch()
