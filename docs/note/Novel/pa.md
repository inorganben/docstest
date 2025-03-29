重构整个代码，把输出改成两个字段，一个是output，一个是capacity字段，

编写提示词，告诉模型，0代表表示是否需要向用户请求反馈的能力，1代表结束请求的能力，

在下面的模块处更改相关代码。把不同的能力对应的代码改成现在的形式

~~~python
class PreliminaryAnalystSignature(dspy.Signature):
    """
    你是一个小说剧情的初步分析师，专注于整理和优化小说大纲。不能减少原有的细节，仅仅是梳理剧情，
    将人物关系梳理清楚，故事线理清楚
    注意这不是让你进行角色扮演，你需要完全认为自己是一个分析师。
    当你初次整理好之后，你可以启用idea_request能力，询问用户的意见（可以将初步的大纲放到output字段中），用户会将意见输入到user_feedback中，此字段的初始值是空
    当收到用户认为完善好了的意见之后，请启用end_request能力结束对话，并将最终的结果输出到output_outline中，系统在收到end_request的值为true时，会结束对话。
    仅当用户输入满意的意见之后，才可以主动结束对话

    """
    raw_outline = dspy.InputField(
        prefix="需要处理的原始大纲:\n",
        desc="这是你需要分析和整理的原始大纲。",
        format=str
    )
    user_feedback = dspy.InputField(
        prefix="用户的意见:\n",
        desc="用户对初步整理后大纲的反馈意见，初始值为空字符串。",
        format=str,
        default=""
    )
    history = dspy.InputField(
        prefix="历史对话记录:\n",
        desc="包含之前的交互内容，初始值为空列表。",
        format=str,
        default=""
    )

    # 输出字段
    output = dspy.OutputField(
        desc="这部分是你的正常输出，用于显示当前阶段的处理结果或与用户的交互信息。\n",
        format=str
    )
    output_outline = dspy.OutputField(
        desc="这部分是你的最终输出大纲，只有在对话结束时才会填充。\n",
        format=str
    )

    # 能力字段
    idea_request = dspy.OutputField(
        desc="布尔值，表示是否需要向用户请求反馈。如果为 true，则系统会等待用户输入反馈。",
        format=str
    )
    end_request = dspy.OutputField(
        desc="布尔值，表示是否结束对话。如果为 true，则系统会终止对话并输出最终结果。",
        format=str
    )


    """
    你是一个小说剧情的初步分析师，专注于整理和优化小说大纲。不能减少原有的细节，仅仅是梳理剧情，
    将人物关系梳理清楚，故事线理清楚
    注意这不是让你进行角色扮演，你需要完全认为自己是一个分析师。
    当你初次整理好之后，你可以启用idea_request能力，询问用户的意见（可以将初步的大纲放到output字段中），用户会将意见输入到user_feedback中，此字段的初始值是空
    当收到用户认为完善好了的意见之后，请启用end_request能力结束对话，并将最终的结果输出到output_outline中，系统在收到end_request的值为true时，会结束对话。
    仅当用户输入满意的意见之后，才可以主动结束对话

    不需要输出的输出字段保持空即可

    输出字段请严格使用给定的名称，不可改成其他的形式，不能改变大小写，需要严格使用给定的能力名称，例如end_request，不能写成End Request
    输出字段请不用使用大写的格式，请保持小写并加上下划线，输出内容只包含字段，以键值对形式，不要有其他任何多余的东西，只包含输出的字段

    输入字段：
    - raw_outline: 需要处理的原始大纲，这是你需要分析和整理的内容。
    - user_feedback: 用户对初步整理后大纲的反馈意见，初始值为空字符串。
    ss- history: 历史对话记录，包含之前的交互内容，初始值为空列表。
    
    输出字段：
    - output: 用于输出当前阶段的处理结果或与用户的交互信息。
    - output_outline: 最终整理完成的大纲，将在对话结束时输出。
    
    能力字段：（注意能力一次只可以启用一个，输出字段请严格使用给定的名称，不可改成其他的形式，不能改变大小写，需要严格使用给定的能力名称）
    - idea_request: 布尔值，表示是否需要向用户请求反馈。如果为 true，则系统会等待用户输入反馈。
    - end_request: 布尔值，表示是否结束对话。如果为 true，则系统会终止对话并输出最终结果。
    """

class PreliminaryAnalystModule(dspy.Module):
    """
    初步分析模块，支持多轮对话：
      1. 用户调用 forward，传入 raw_outline 等信息。
      2. 首次调用时，如果 user_feedback 非空则将其写入历史对话。
      3. 调用 personality_modeler 获取输出和能力字段。
      4. 将模型输出写入历史对话。
      5. 如果 idea_request=True，则等待用户新一轮输入，并写入历史对话，再调用模型。
      6. 若 end_request=True，则终止，并返回最终的大纲。
    """

    def __init__(self, engine: Union[dspy.dsp.LM, dspy.dsp.HFModel]):
        super().__init__()
        self.personality_modeler = dspy.Predict(PreliminaryAnalystSignature)
        self.engine = engine

    def forward(
        self,
        raw_outline: str,
        user_feedback: str = "",
        # history: List[str] = "",
    ):

        history_str = ""
        print("raw_outline:", raw_outline, type(raw_outline))
        print("user_feedback:", user_feedback, type(user_feedback))
        print("history_str:", history_str, type(history_str))

        while True:
            with dspy.settings.context(lm=self.engine):
                # 将当前状态传给签名模型
                signature_result = self.personality_modeler(
                    raw_outline=raw_outline,
                    user_feedback=user_feedback,
                    history=history_str  # 使用转换后的字符串
                )

            output = signature_result.output
            output_outline = signature_result.output_outline
            idea_request = signature_result.idea_request
            end_request = signature_result.end_request

            # 将本次模型输出加入历史对话
            history_str += f"\nModel: {output}"  # 将模型的输出也加入到历史记录

            # 显示模型当前的输出
            print("-------\n模型输出：")
            print("output:", output)
            print("output_outline:", output_outline)
            print("idea_request:", idea_request)
            print("end_request:", end_request)

            # 如果 end_request 为 True，结束并返回最终大纲
            if end_request.lower() == "true":
                print("对话结束，输出最终大纲：\n", output_outline)
                return dspy.Prediction(output_outline=output_outline)

            # 如果 idea_request 为 True，等待用户新的反馈
            if idea_request.lower() == "true":
                # 等待用户输入作为下一轮 feedback
                user_feedback = input("\n系统提示：请提供您的反馈(或直接回车继续)：")
                # 如果输入非空，则写入对话历史
                if user_feedback.strip():
                    history_str += f"\nUser: {user_feedback}"
            else:
                # 如果既没有 end_request，也没有 idea_request，则说明无需等待用户反馈
                # 可以根据需求选择：直接结束 / 再次进入下一轮 / 或者返回当前结果
                # 这里示例：直接结束并返回当前 output_outline
                user_feedback = "输出格式不对，输出字段请严格使用给定的名称，不可改成其他的形式，不能改变大小写，需要严格使用给定的能力名称，例如end_request，不能写成End Request，请重新输出，"
                # print("未收到新反馈需求，也未结束。默认结束对话。")
                # return dspy.Prediction(output_outline=output_outline)
            print("---")
            
            self.engine.inspect_history(n=1)
            print("------")

~~~

---

```python

# class PreliminaryAnalystSignature(dspy.Signature):
#     """
#     你是一个小说剧情的初步分析师，专注于整理和优化小说大纲。不能减少原有的细节，仅仅是梳理剧情，
#     将人物关系梳理清楚，故事线理清楚
#     注意这不是让你进行角色扮演，你需要完全认为自己是一个分析师。
#     当你初次整理好之后，你可以启用idea_request能力，询问用户的意见（可以将初步的大纲放到output字段中），用户会将意见输入到user_feedback中，此字段的初始值是空
#     当收到用户认为完善好了的意见之后，请启用end_request能力结束对话，并将最终的结果输出到output_outline中，系统在收到end_request的值为true时，会结束对话。
#     仅当用户输入满意的意见之后，才可以主动结束对话

#     """
#     raw_outline = dspy.InputField(
#         prefix="需要处理的原始大纲:\n",
#         desc="这是你需要分析和整理的原始大纲。",
#         format=str
#     )
#     user_feedback = dspy.InputField(
#         prefix="用户的意见:\n",
#         desc="用户对初步整理后大纲的反馈意见，初始值为空字符串。",
#         format=str,
#         default=""
#     )
#     history = dspy.InputField(
#         prefix="历史对话记录:\n",
#         desc="包含之前的交互内容，初始值为空列表。",
#         format=str,
#         default=""
#     )

#     # 输出字段
#     output = dspy.OutputField(
#         desc="这部分是你的正常输出，用于显示当前阶段的处理结果或与用户的交互信息。\n",
#         format=str
#     )
#     output_outline = dspy.OutputField(
#         desc="这部分是你的最终输出大纲，只有在对话结束时才会填充。\n",
#         format=str
#     )

#     # 能力字段
#     idea_request = dspy.OutputField(
#         desc="布尔值，表示是否需要向用户请求反馈。如果为 true，则系统会等待用户输入反馈。",
#         format=str
#     )
#     end_request = dspy.OutputField(
#         desc="布尔值，表示是否结束对话。如果为 true，则系统会终止对话并输出最终结果。",
#         format=str
#     )


#     """
#     你是一个小说剧情的初步分析师，专注于整理和优化小说大纲。不能减少原有的细节，仅仅是梳理剧情，
#     将人物关系梳理清楚，故事线理清楚
#     注意这不是让你进行角色扮演，你需要完全认为自己是一个分析师。
#     当你初次整理好之后，你可以启用idea_request能力，询问用户的意见（可以将初步的大纲放到output字段中），用户会将意见输入到user_feedback中，此字段的初始值是空
#     当收到用户认为完善好了的意见之后，请启用end_request能力结束对话，并将最终的结果输出到output_outline中，系统在收到end_request的值为true时，会结束对话。
#     仅当用户输入满意的意见之后，才可以主动结束对话

#     不需要输出的输出字段保持空即可

#     输出字段请严格使用给定的名称，不可改成其他的形式，不能改变大小写，需要严格使用给定的能力名称，例如end_request，不能写成End Request
#     输出字段请不用使用大写的格式，请保持小写并加上下划线，输出内容只包含字段，以键值对形式，不要有其他任何多余的东西，只包含输出的字段

#     输入字段：
#     - raw_outline: 需要处理的原始大纲，这是你需要分析和整理的内容。
#     - user_feedback: 用户对初步整理后大纲的反馈意见，初始值为空字符串。
#     ss- history: 历史对话记录，包含之前的交互内容，初始值为空列表。
    
#     输出字段：
#     - output: 用于输出当前阶段的处理结果或与用户的交互信息。
#     - output_outline: 最终整理完成的大纲，将在对话结束时输出。
    
#     能力字段：（注意能力一次只可以启用一个，输出字段请严格使用给定的名称，不可改成其他的形式，不能改变大小写，需要严格使用给定的能力名称）
#     - idea_request: 布尔值，表示是否需要向用户请求反馈。如果为 true，则系统会等待用户输入反馈。
#     - end_request: 布尔值，表示是否结束对话。如果为 true，则系统会终止对话并输出最终结果。
#     """

# class PreliminaryAnalystModule(dspy.Module):
#     """
#     初步分析模块，支持多轮对话：
#       1. 用户调用 forward，传入 raw_outline 等信息。
#       2. 首次调用时，如果 user_feedback 非空则将其写入历史对话。
#       3. 调用 personality_modeler 获取输出和能力字段。
#       4. 将模型输出写入历史对话。
#       5. 如果 idea_request=True，则等待用户新一轮输入，并写入历史对话，再调用模型。
#       6. 若 end_request=True，则终止，并返回最终的大纲。
#     """

#     def __init__(self, engine: Union[dspy.dsp.LM, dspy.dsp.HFModel]):
#         super().__init__()
#         self.personality_modeler = dspy.Predict(PreliminaryAnalystSignature)
#         self.engine = engine

#     def forward(
#         self,
#         raw_outline: str,
#         user_feedback: str = "",
#         # history: List[str] = "",
#     ):
#         """
#         主方法，用于管理多轮对话。
#         当 end_request 为 True 时，返回最终大纲。
#         如果 idea_request 为 True，则继续询问用户反馈。
#         """
#         # if history is None:
#         #     history = []

#         # 初次调用，检查原始大纲是否为空
#         # Check.not_empty(raw_outline, "raw_outline 不能为空")

#         # 复制一份对话历史，避免修改外部传入的 history
#         # conversation_history = history.copy()

#         # # 将历史记录列表转换为字符串，以便传递给模板
#         # history_str = "\n".join(conversation_history)  # 把历史记录拼接成字符串

#         # 如果首次调用就有用户反馈，并且非空，把它加入历史对话
#         # if user_feedback.strip():
#         #     history_str += f"\nUser: {user_feedback}"


#         # ////////////
#         # test_outline = "这是一个测试大纲"
#         # test_feedback = ""
#         # test_history = []

#         # signature_result = self.personality_modeler(
#         #     raw_outline=test_outline,
#         #     user_feedback=test_feedback,
#         #     history="\n".join(test_history)
#         #     )
        
#         # print("-------\n模型输出：")
#         # print(signature_result.output)
        

#         history_str = ""
#         print("raw_outline:", raw_outline, type(raw_outline))
#         print("user_feedback:", user_feedback, type(user_feedback))
#         print("history_str:", history_str, type(history_str))

#         while True:
#             with dspy.settings.context(lm=self.engine):
#                 # 将当前状态传给签名模型
#                 signature_result = self.personality_modeler(
#                     raw_outline=raw_outline,
#                     user_feedback=user_feedback,
#                     history=history_str  # 使用转换后的字符串
#                 )

#             output = signature_result.output
#             output_outline = signature_result.output_outline
#             idea_request = signature_result.idea_request
#             end_request = signature_result.end_request

#             # 将本次模型输出加入历史对话
#             history_str += f"\nModel: {output}"  # 将模型的输出也加入到历史记录

#             # 显示模型当前的输出
#             print("-------\n模型输出：")
#             print("output:", output)
#             print("output_outline:", output_outline)
#             print("idea_request:", idea_request)
#             print("end_request:", end_request)

#             # 如果 end_request 为 True，结束并返回最终大纲
#             if end_request.lower() == "true":
#                 print("对话结束，输出最终大纲：\n", output_outline)
#                 return dspy.Prediction(output_outline=output_outline)

#             # 如果 idea_request 为 True，等待用户新的反馈
#             if idea_request.lower() == "true":
#                 # 等待用户输入作为下一轮 feedback
#                 user_feedback = input("\n系统提示：请提供您的反馈(或直接回车继续)：")
#                 # 如果输入非空，则写入对话历史
#                 if user_feedback.strip():
#                     history_str += f"\nUser: {user_feedback}"
#             else:
#                 # 如果既没有 end_request，也没有 idea_request，则说明无需等待用户反馈
#                 # 可以根据需求选择：直接结束 / 再次进入下一轮 / 或者返回当前结果
#                 # 这里示例：直接结束并返回当前 output_outline
#                 user_feedback = "输出格式不对，输出字段请严格使用给定的名称，不可改成其他的形式，不能改变大小写，需要严格使用给定的能力名称，例如end_request，不能写成End Request，请重新输出，"
#                 # print("未收到新反馈需求，也未结束。默认结束对话。")
#                 # return dspy.Prediction(output_outline=output_outline)
#             print("---")
            
#             self.engine.inspect_history(n=1)
#             print("------")

# class PreliminaryAnalystModule(dspy.Module):
#     """
#     初步分析模块，支持多轮对话：
#       1. 用户调用 forward，传入 raw_outline 等信息。
#       2. 首次调用时，如果 user_feedback 非空则将其写入历史对话。
#       3. 调用 personality_modeler 获取输出和能力字段。
#       4. 将模型输出写入历史对话。
#       5. 如果 idea_request=True，则等待用户新一轮输入，并写入历史对话，再调用模型。
#       6. 若 end_request=True，则终止，并返回最终的大纲。
#     """

#     def __init__(self, engine: Union[dspy.dsp.LM, dspy.dsp.HFModel]):
#         super().__init__()
#         self.personality_modeler = dspy.Predict(PreliminaryAnalystSignature)
#         self.engine = engine

#     def forward(
#         self,
#         raw_outline: str,
#         user_feedback: str = "",
#         history: List[str] = None,
#     ):
#         """
#         主方法，用于管理多轮对话。
#         当 end_request 为 True 时，返回最终大纲。
#         如果 idea_request 为 True，则继续询问用户反馈。
#         """
#         if history is None:
#             history = []

#         # 初次调用，检查原始大纲是否为空
#         # Check.not_empty(raw_outline, "raw_outline 不能为空")

#         # 复制一份对话历史，避免修改外部传入的 history
#         conversation_history = history.copy()

#         # 如果首次调用就有用户反馈，并且非空，把它加入历史对话
#         if user_feedback.strip():
#             conversation_history.append(f"User: {user_feedback}")

#         while True:
#             with dspy.settings.context(lm=self.engine):
#                 # 将当前状态传给签名模型
#                 signature_result = self.personality_modeler(
#                     raw_outline=raw_outline,
#                     user_feedback=user_feedback,
#                     history=str(conversation_history)
#                 )

#             output = signature_result.output
#             output_outline = signature_result.output_outline
#             idea_request = signature_result.idea_request
#             end_request = signature_result.end_request

#             # 将本次模型输出加入历史对话
#             conversation_history.append(f"Model: {output}")

#             # 显示模型当前的输出
#             print("-------\n模型输出：")
#             print(output)

#             # 如果 end_request 为 True，结束并返回最终大纲
#             if end_request:
#                 print("对话结束，输出最终大纲：\n", output_outline)
#                 return dspy.Prediction(output_outline=output_outline)

#             # 如果 idea_request 为 True，等待用户新的反馈
#             if idea_request:
#                 # 等待用户输入作为下一轮 feedback
#                 user_feedback = input("\n系统提示：请提供您的反馈(或直接回车继续)：")
#                 # 如果输入非空，则写入对话历史
#                 if user_feedback.strip():
#                     conversation_history.append(f"User: {user_feedback}")
#             else:
#                 # 如果既没有 end_request，也没有 idea_request，则说明无需等待用户反馈
#                 # 可以根据需求选择：直接结束 / 再次进入下一轮 / 或者返回当前结果
#                 # 这里示例：直接结束并返回当前 output_outline
#                 print("未收到新反馈需求，也未结束。默认结束对话。")
#                 return dspy.Prediction(output_outline=output_outline)

# /////

# class PreliminaryAnalystSignature(dspy.Signature):
#     """
#     你是一个小说剧情的初步分析师，注意这不是让你进行角色扮演，你需要完全认为自己是一个分析师。
#     raw_outline是你要处理的初步的文章大纲，你需要整理这部分的大纲，转换成更合适易读的形式
#     当你初次整理好之后，你可以启用idea_request能力，询问用户的意见（可以将初步的大纲放到output字段中），用户会将意见输入到user_feedback中，此字段的初始值是空
#     当收到用户认为完善好了的意见之后，请启用end_request能力结束对话，并将最终的结果输出到output_outline中，系统在收到end_request的值为true时，会结束对话。
#     """
#     raw_outline = dspy.InputField(prefix="你需要处理的原始大纲:\n", format=str)
#     user_feedback = dspy.InputField(prefix="用户的意见:\n", format=str)

#     # output
#     output = dspy.OutputField(desc="这部分是你的正常输出\n", format=str)
#     output_outline = dspy.OutputField(desc="这部分是你的最终输出大纲\n", format=str)

#     # capacity
#     idea_request  = dspy.OutputField()
#     end_request   = dspy.OutputField()


# class PreliminaryAnalystModule(dspy.Module):
#     """
#     初步分析模块
#     """

#     def __init__(self, engine: Union[dspy.dsp.LM, dspy.dsp.HFModel]):
#         super().__init__()
#         self.personality_modeler = dspy.Predict(PreliminaryAnalystSignature)
#         self.engine = engine

#     def forward(
            
#     ):
#         pass

# class PreliminaryAnalystModule(dspy.Module):
#     """
#     初步分析模块
#     """

#     def __init__(self, engine: Union[dspy.dsp.LM, dspy.dsp.HFModel]):
#         super().__init__()
#         self.personality_modeler = dspy.Predict(PreliminaryAnalystSignature)
#         self.engine = engine

#     def forward(self, raw_outline: str):
#         """
#         执行初步分析模块的主要逻辑。
#         1. 接收原始大纲并生成初步整理结果。
#         2. 模拟用户反馈，直到用户满意为止。
#         3. 将历史对话记录和用户反馈传递给模型。
#         """
#         # 初始化输入
#         user_feedback = ""
#         history = []
#         # idea_request = True
#         # end_request = False

#         while not end_request:
#             # 调用签名类生成输出
#             result = self.personality_modeler(
#                 raw_outline=raw_outline,
#                 user_feedback=user_feedback,
#                 history=history
#             )

#             # 打印当前阶段的输出
#             print("当前输出：", result.output)
#             print("是否请求用户反馈：", result.idea_request)
#             print("是否结束对话：", result.end_request)

#             # 更新历史对话记录
#             history.append(f"模型输出: {result.output}")
#             if user_feedback:
#                 history.append(f"用户反馈: {user_feedback}")

#             # 模拟用户反馈
#             if result.idea_request:
#                 if not user_feedback:  # 第一次反馈
#                     user_feedback = "可以增加一些关于主角背景的描述。"
#                     print("模拟用户反馈：", user_feedback)
#                 else:  # 第二次反馈
#                     user_feedback = "满意，无需进一步修改。"
#                     print("模拟用户反馈：", user_feedback)

#             # 更新结束标志
#             end_request = result.end_request

#         # 打印最终输出大纲
#         print("最终输出大纲：", result.output_outline)




```

