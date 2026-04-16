
- **Phase 0: Sketch It Out** (Visualizing the bridge between your head and logic) 
	**阶段 0：草绘构思**（将脑海中的想法与逻辑之间的桥梁可视化）
	![[p0_visualization.png]]
    - Home Screen: Project Cards
    - Detail Page: Project Details
    - Admin Login
    - ...
    Fixed the mistake of humans thinking visually while AI by text
- **Phase 1: Talk to the AI Like a Colleague** (Briefing, not begging) **阶段 1：像对待同事一样与 AI 沟通**（下达简报，而不是乞求）
	- Explain Your Problem

	> I want a personal portfolio website where I can show my projects. Visitors browse through them, see images and descriptions. I can add new projects through a protected admin area. Simple — no overkill, no complex CMS." My portfolio app has five screens: A homepage with project cards. A detail page per project. An admin login. An admin dashboard listing all projects. An admin form for creating new ones with image upload.
	> anything else in mind? any suggestions? let's just talk. don't code anything.
	
	**WHAT I AM BUILDING**
	**WHAT USER ARE EXPECTED TO DO**
	**WHAT ADMIN CAN DO**
	**CONSTRAITS**
	**LIST ALL SCREENS**

anything else in mind? any suggestions?
    
- **Phase 2: Let the AI Suggest the Tech Stack** (Leveraging "high-density" training data) **阶段 2：让 AI 建议技术栈**（利用其“高密度”的训练数据）
    
- **Phase 3: Split Everything into Features** (The Lego-brick approach) **阶段 3：将所有内容拆分为功能点**（乐高积木式方法）
    
- **Phase 4: Ask for Best Practices and Patterns** (Setting the architectural direction) **阶段 4：询问最佳实践和设计模式**（确定架构方向）
    
- **Phase 5: Validate the Data Model** (Ensuring the foundation is solid) **阶段 5：验证数据模型**（确保基础稳固）
    
- **Phase 6: Have the AI Write a Spec** (Creating the single source of truth) **阶段 6：让 AI 编写规格说明书**（创建唯一的真相来源/标准文档）
    
- **Phase 7: Read the Spec and Adjust** (Playing the Project Manager) **阶段 7：阅读规格书并进行调整**（扮演项目经理的角色）
    
- **Phase 8: Fresh Review** (The "Second Opinion" technique) **阶段 8：全新审查**（采用“第二意见”技术）
    
- **Phase 9: Environment Setup** (Overcoming the "it doesn't run" hurdle) **阶段 9：环境搭建**（克服“无法运行”的障碍）
    
- **Phase 10: Build Feature by Feature** (Atomic development and testing) **阶段 10：逐个功能进行构建**（原子级开发与测试）
    
- **Phase 11: Debugging is Normal** (The Where-What-Error formula) **阶段 11：调试是常态**（遵循“何处-何事-错误”公式）
    
- **Phase 12: Code Review by a Second AI** (Automated quality assurance) **阶段 12：由第二个 AI 进行代码审查**（自动化质量保证）

## Phase 1 实践

I'm building a new 'Daily Practice' feature for my language app. The system currently allows users to chat with an AI partner, polish their messages, and save expressions. This new feature will use an SRS mechanism to pick a target word and fetch scenario-based exercises from an LLM. I'll need to handle JSON data storage for these scenarios and implement a 'Word Bank' bonus system. Here is the breakdown of the feature, user interactions, and UI logic.