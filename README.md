<!doctype html>
<html lang="en" class="h-full">
 <head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Algorithm Repetition Master</title>
  <script src="/_sdk/element_sdk.js"></script>
  <script src="/_sdk/data_sdk.js"></script>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body {
      box-sizing: border-box;
    }
    
    @keyframes pulse-glow {
      0%, 100% { box-shadow: 0 0 20px rgba(139, 92, 246, 0.5); }
      50% { box-shadow: 0 0 40px rgba(139, 92, 246, 0.8); }
    }
    
    @keyframes float {
      0%, 100% { transform: translateY(0px); }
      50% { transform: translateY(-10px); }
    }
    
    @keyframes checkmark {
      0% { stroke-dashoffset: 100; }
      100% { stroke-dashoffset: 0; }
    }
    
    @keyframes celebration {
      0%, 100% { transform: scale(1) rotate(0deg); }
      25% { transform: scale(1.1) rotate(5deg); }
      75% { transform: scale(1.1) rotate(-5deg); }
    }
    
    .animate-pulse-glow {
      animation: pulse-glow 2s ease-in-out infinite;
    }
    
    .animate-float {
      animation: float 3s ease-in-out infinite;
    }
    
    .animate-checkmark {
      animation: checkmark 0.5s ease-out forwards;
    }
    
    .animate-celebration {
      animation: celebration 0.5s ease-in-out;
    }
    
    .flowchart-box {
      fill: white;
      stroke: #6366f1;
      stroke-width: 2;
    }
    
    .flowchart-diamond {
      fill: #fef3c7;
      stroke: #f59e0b;
      stroke-width: 2;
    }
    
    .flowchart-text {
      font-family: 'Courier New', monospace;
      font-size: 12px;
      fill: #1f2937;
    }
    
    .flowchart-arrow {
      stroke: #6366f1;
      stroke-width: 2;
      fill: none;
      marker-end: url(#arrowhead);
    }
  </style>
  <style>@view-transition { navigation: auto; }</style>
 </head>
 <body class="h-full">
  <div id="app" class="w-full h-full overflow-auto"></div>
  <script>
    const defaultConfig = {
      background_color: "#0f172a",
      surface_color: "#1e293b",
      text_color: "#f1f5f9",
      primary_action_color: "#8b5cf6",
      secondary_action_color: "#06b6d4",
      font_family: "Inter",
      font_size: 16,
      main_title: "Algorithm Repetition Master",
      subtitle: "Master Loops & Repetition Structures - Form One Computing",
      learn_button_text: "📚 Learn Concepts",
      practice_button_text: "✏️ Practice Flowcharts",
      quiz_button_text: "🎮 Take Quiz",
      leaderboard_button_text: "🏆 Leaderboard"
    };

    let currentView = 'home';
    let quizState = {
      currentQuestion: 0,
      score: 0,
      answers: [],
      studentName: ''
    };
    
    let records = [];

    const quizQuestions = [
      {
        question: "What is a repetition structure in algorithms?",
        options: [
          "A way to make decisions in code",
          "A control structure that repeats a set of instructions",
          "A method to store data",
          "A way to define variables"
        ],
        correct: 1,
        explanation: "A repetition structure (or loop) allows a set of instructions to be executed multiple times."
      },
      {
        question: "Which pseudocode keyword indicates repetition?",
        options: ["IF", "WHILE", "INPUT", "OUTPUT"],
        correct: 1,
        explanation: "WHILE is a common keyword for repetition that continues as long as a condition is true."
      },
      {
        question: "In a flowchart, which shape represents a decision/condition?",
        options: ["Rectangle", "Diamond", "Oval", "Parallelogram"],
        correct: 1,
        explanation: "A diamond shape represents a decision or condition that can be true or false."
      },
      {
        question: "What does 'REPEAT UNTIL condition' mean?",
        options: [
          "Repeat once then stop",
          "Repeat forever",
          "Keep repeating until the condition becomes TRUE",
          "Never repeat"
        ],
        correct: 2,
        explanation: "REPEAT UNTIL continues looping until the specified condition becomes TRUE."
      },
      {
        question: "Which is the correct pseudocode for counting from 1 to 5?",
        options: [
          "SET count TO 1\nIF count ≤ 5 THEN\nOUTPUT count",
          "SET count TO 1\nWHILE count ≤ 5 DO\nOUTPUT count\ncount ← count + 1\nEND WHILE",
          "SET count TO 5\nOUTPUT count",
          "REPEAT 1 TO 5"
        ],
        correct: 1,
        explanation: "This uses a WHILE loop that outputs the count and increments it until reaching 5."
      },
      {
        question: "What is an infinite loop?",
        options: [
          "A loop that runs very fast",
          "A loop that never terminates",
          "A loop that runs exactly 10 times",
          "A loop that only runs once"
        ],
        correct: 1,
        explanation: "An infinite loop never stops because its termination condition is never met."
      },
      {
        question: "In flowcharts, what do arrows represent?",
        options: [
          "Data storage",
          "Flow of control/direction",
          "Input operations",
          "Output operations"
        ],
        correct: 1,
        explanation: "Arrows show the direction and flow of control through the algorithm."
      },
      {
        question: "What shape is used for the start/end in a flowchart?",
        options: ["Rectangle", "Diamond", "Oval/Rounded rectangle", "Parallelogram"],
        correct: 2,
        explanation: "Ovals or rounded rectangles indicate the start and end points of a flowchart."
      }
    ];

    const dataHandler = {
      onDataChanged(data) {
        records = data;
        if (currentView === 'leaderboard') {
          renderLeaderboard();
        }
      }
    };

    async function initializeApp() {
      const initResult = await window.dataSdk.init(dataHandler);
      if (!initResult.isOk) {
        console.error("Failed to initialize data SDK");
      }
      
      if (window.elementSdk) {
        window.elementSdk.init({
          defaultConfig,
          onConfigChange: async (config) => {
            Object.assign(defaultConfig, config);
            renderCurrentView();
          },
          mapToCapabilities: (config) => ({
            recolorables: [
              {
                get: () => config.background_color || defaultConfig.background_color,
                set: (value) => {
                  defaultConfig.background_color = value;
                  window.elementSdk.setConfig({ background_color: value });
                }
              },
              {
                get: () => config.surface_color || defaultConfig.surface_color,
                set: (value) => {
                  defaultConfig.surface_color = value;
                  window.elementSdk.setConfig({ surface_color: value });
                }
              },
              {
                get: () => config.text_color || defaultConfig.text_color,
                set: (value) => {
                  defaultConfig.text_color = value;
                  window.elementSdk.setConfig({ text_color: value });
                }
              },
              {
                get: () => config.primary_action_color || defaultConfig.primary_action_color,
                set: (value) => {
                  defaultConfig.primary_action_color = value;
                  window.elementSdk.setConfig({ primary_action_color: value });
                }
              },
              {
                get: () => config.secondary_action_color || defaultConfig.secondary_action_color,
                set: (value) => {
                  defaultConfig.secondary_action_color = value;
                  window.elementSdk.setConfig({ secondary_action_color: value });
                }
              }
            ],
            borderables: [],
            fontEditable: {
              get: () => config.font_family || defaultConfig.font_family,
              set: (value) => {
                defaultConfig.font_family = value;
                window.elementSdk.setConfig({ font_family: value });
              }
            },
            fontSizeable: {
              get: () => config.font_size || defaultConfig.font_size,
              set: (value) => {
                defaultConfig.font_size = value;
                window.elementSdk.setConfig({ font_size: value });
              }
            }
          }),
          mapToEditPanelValues: (config) => new Map([
            ["main_title", config.main_title || defaultConfig.main_title],
            ["subtitle", config.subtitle || defaultConfig.subtitle],
            ["learn_button_text", config.learn_button_text || defaultConfig.learn_button_text],
            ["practice_button_text", config.practice_button_text || defaultConfig.practice_button_text],
            ["quiz_button_text", config.quiz_button_text || defaultConfig.quiz_button_text],
            ["leaderboard_button_text", config.leaderboard_button_text || defaultConfig.leaderboard_button_text]
          ])
        });
      }
      
      renderHome();
    }

    function renderCurrentView() {
      switch(currentView) {
        case 'home': renderHome(); break;
        case 'learn': renderLearn(); break;
        case 'practice': renderPractice(); break;
        case 'quiz': renderQuiz(); break;
        case 'leaderboard': renderLeaderboard(); break;
      }
    }

    function renderHome() {
      currentView = 'home';
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const fontSize = config.font_size || defaultConfig.font_size;
      
      document.getElementById('app').innerHTML = `
        <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px;">
          <div style="max-width: ${fontSize * 50}px; margin: 0 auto;">
            <!-- Header -->
            <div style="text-align: center; margin-bottom: ${fontSize * 3}px;">
              <div class="animate-float" style="font-size: ${fontSize * 4}px; margin-bottom: ${fontSize}px;">🔄</div>
              <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2.5}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                ${config.main_title || defaultConfig.main_title}
              </h1>
              <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 1.1}px;">
                ${config.subtitle || defaultConfig.subtitle}
              </p>
            </div>

            <!-- Menu Cards -->
            <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(${fontSize * 15}px, 1fr)); gap: ${fontSize * 1.5}px; margin-bottom: ${fontSize * 2}px;">
              <!-- Learn Card -->
              <button onclick="navigateTo('learn')" style="background: ${config.surface_color || defaultConfig.surface_color}; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; cursor: pointer; transition: all 0.3s; text-align: left;">
                <div style="font-size: ${fontSize * 3}px; margin-bottom: ${fontSize}px;">📚</div>
                <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  ${config.learn_button_text || defaultConfig.learn_button_text}
                </h3>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.7; font-size: ${fontSize * 0.9}px;">
                  Learn about WHILE loops, REPEAT UNTIL, and flowchart symbols
                </p>
              </button>

              <!-- Practice Card -->
              <button onclick="navigateTo('practice')" style="background: ${config.surface_color || defaultConfig.surface_color}; border: 2px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; cursor: pointer; transition: all 0.3s; text-align: left;">
                <div style="font-size: ${fontSize * 3}px; margin-bottom: ${fontSize}px;">✏️</div>
                <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  ${config.practice_button_text || defaultConfig.practice_button_text}
                </h3>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.7; font-size: ${fontSize * 0.9}px;">
                  Interactive flowchart examples and pseudocode practice
                </p>
              </button>

              <!-- Quiz Card -->
              <button onclick="navigateTo('quiz')" style="background: ${config.surface_color || defaultConfig.surface_color}; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; cursor: pointer; transition: all 0.3s; text-align: left;">
                <div style="font-size: ${fontSize * 3}px; margin-bottom: ${fontSize}px;">🎮</div>
                <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  ${config.quiz_button_text || defaultConfig.quiz_button_text}
                </h3>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.7; font-size: ${fontSize * 0.9}px;">
                  Test your knowledge with interactive questions
                </p>
              </button>

              <!-- Leaderboard Card -->
              <button onclick="navigateTo('leaderboard')" style="background: ${config.surface_color || defaultConfig.surface_color}; border: 2px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; cursor: pointer; transition: all 0.3s; text-align: left;">
                <div style="font-size: ${fontSize * 3}px; margin-bottom: ${fontSize}px;">🏆</div>
                <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  ${config.leaderboard_button_text || defaultConfig.leaderboard_button_text}
                </h3>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.7; font-size: ${fontSize * 0.9}px;">
                  View top scores and challenge yourself
                </p>
              </button>
            </div>

            <!-- Info Box -->
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-left: 4px solid ${config.primary_action_color || defaultConfig.primary_action_color}; border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.5}px; margin-top: ${fontSize * 2}px;">
              <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize}px; line-height: 1.6;">
                <strong>Learning Objectives:</strong><br>
                ✓ Explain repetition structures in algorithms<br>
                ✓ Write pseudocode using WHILE and REPEAT UNTIL<br>
                ✓ Draw flowcharts with correct repetition symbols
              </p>
            </div>
          </div>
        </div>
      `;
    }

    function renderLearn() {
      currentView = 'learn';
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const fontSize = config.font_size || defaultConfig.font_size;
      
      document.getElementById('app').innerHTML = `
        <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px;">
          <div style="max-width: ${fontSize * 56}px; margin: 0 auto;">
            <button onclick="navigateTo('home')" style="background: ${config.surface_color || defaultConfig.surface_color}; color: ${config.text_color || defaultConfig.text_color}; border: 1px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 0.75}px ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; margin-bottom: ${fontSize * 2}px; font-size: ${fontSize}px;">
              ← Back to Home
            </button>

            <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2.5}px; font-weight: bold; margin-bottom: ${fontSize * 2}px;">
              📚 Learn About Repetition Structures
            </h1>

            <!-- Simple Explanation Box -->
            <div style="background: linear-gradient(135deg, rgba(139, 92, 246, 0.15), rgba(6, 182, 212, 0.15)); border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2.5}px; margin-bottom: ${fontSize * 2}px; border: 3px solid ${config.primary_action_color || defaultConfig.primary_action_color};">
              <div style="text-align: center; margin-bottom: ${fontSize * 1.5}px;">
                <div style="font-size: ${fontSize * 3.5}px; margin-bottom: ${fontSize * 0.5}px;">🔁</div>
                <h2 style="color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-size: ${fontSize * 2}px; font-weight: bold;">
                  Repetition Made Simple!
                </h2>
              </div>
              
              <div style="background: white; border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.75}px; margin-bottom: ${fontSize * 1.5}px;">
                <h3 style="color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-size: ${fontSize * 1.4}px; font-weight: bold; margin-bottom: ${fontSize * 0.75}px;">
                  🤔 Think of it like this:
                </h3>
                <p style="color: #1f2937; font-size: ${fontSize * 1.1}px; line-height: 1.8; margin-bottom: ${fontSize}px;">
                  Imagine you need to <strong>count from 1 to 10</strong>. You could write:<br>
                  <span style="background: #fee2e2; padding: ${fontSize * 0.25}px ${fontSize * 0.5}px; border-radius: ${fontSize * 0.25}px;">
                    Say "1", Say "2", Say "3"... Say "10"
                  </span> ❌ <em>(Way too long!)</em>
                </p>
                <p style="color: #1f2937; font-size: ${fontSize * 1.1}px; line-height: 1.8;">
                  <strong>OR</strong> use a loop: <br>
                  <span style="background: #dcfce7; padding: ${fontSize * 0.25}px ${fontSize * 0.5}px; border-radius: ${fontSize * 0.25}px;">
                    "Keep saying the next number UNTIL you reach 10"
                  </span> ✅ <em>(Much smarter!)</em>
                </p>
              </div>

              <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(${fontSize * 14}px, 1fr)); gap: ${fontSize * 1.25}px; margin-bottom: ${fontSize * 1.5}px;">
                <div style="background: #dbeafe; border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.5}px; border: 2px solid #3b82f6;">
                  <div style="font-size: ${fontSize * 2.5}px; margin-bottom: ${fontSize * 0.5}px; text-align: center;">🍪</div>
                  <h4 style="color: #1e40af; font-size: ${fontSize * 1.15}px; font-weight: bold; margin-bottom: ${fontSize * 0.75}px; text-align: center;">
                    Baking Cookies
                  </h4>
                  <p style="color: #1e3a8a; font-size: ${fontSize * 0.95}px; line-height: 1.6;">
                    <strong>WHILE</strong> there is cookie dough:<br>
                    &nbsp;&nbsp;• Take some dough<br>
                    &nbsp;&nbsp;• Make a cookie<br>
                    &nbsp;&nbsp;• Put it in the oven<br>
                    <em style="font-size: ${fontSize * 0.85}px;">(Keeps going until dough runs out)</em>
                  </p>
                </div>

                <div style="background: #fef3c7; border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.5}px; border: 2px solid #f59e0b;">
                  <div style="font-size: ${fontSize * 2.5}px; margin-bottom: ${fontSize * 0.5}px; text-align: center;">🎮</div>
                  <h4 style="color: #92400e; font-size: ${fontSize * 1.15}px; font-weight: bold; margin-bottom: ${fontSize * 0.75}px; text-align: center;">
                    Playing a Game
                  </h4>
                  <p style="color: #78350f; font-size: ${fontSize * 0.95}px; line-height: 1.6;">
                    <strong>REPEAT</strong><br>
                    &nbsp;&nbsp;• Try to score<br>
                    &nbsp;&nbsp;• Check result<br>
                    <strong>UNTIL</strong> you win!<br>
                    <em style="font-size: ${fontSize * 0.85}px;">(Keeps trying until success)</em>
                  </p>
                </div>

                <div style="background: #dcfce7; border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.5}px; border: 2px solid #22c55e;">
                  <div style="font-size: ${fontSize * 2.5}px; margin-bottom: ${fontSize * 0.5}px; text-align: center;">🧺</div>
                  <h4 style="color: #166534; font-size: ${fontSize * 1.15}px; font-weight: bold; margin-bottom: ${fontSize * 0.75}px; text-align: center;">
                    Doing Laundry
                  </h4>
                  <p style="color: #14532d; font-size: ${fontSize * 0.95}px; line-height: 1.6;">
                    <strong>WHILE</strong> clothes are dirty:<br>
                    &nbsp;&nbsp;• Wash clothes<br>
                    &nbsp;&nbsp;• Dry clothes<br>
                    &nbsp;&nbsp;• Fold clothes<br>
                    <em style="font-size: ${fontSize * 0.85}px;">(Stops when all clean)</em>
                  </p>
                </div>
              </div>

              <div style="background: rgba(139, 92, 246, 0.2); border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.5}px; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color};">
                <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.05}px; line-height: 1.7; margin: 0;">
                  <strong style="font-size: ${fontSize * 1.2}px;">💡 Key Idea:</strong><br>
                  Instead of writing the same instruction over and over, we tell the computer to <strong>repeat</strong> those instructions automatically! This saves time and makes our algorithms much shorter and smarter.
                </p>
              </div>
            </div>

            <!-- Concept 1 -->
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color};">
              <h2 style="color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-size: ${fontSize * 1.75}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                What is Repetition?
              </h2>
              <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.1}px; line-height: 1.6; margin-bottom: ${fontSize}px;">
                Repetition (also called <strong>iteration</strong> or <strong>looping</strong>) is when a set of instructions is executed multiple times. This is one of the three basic control structures in programming.
              </p>
              <div style="background: ${config.background_color || defaultConfig.background_color}; padding: ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; margin-top: ${fontSize}px;">
                <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize}px; font-family: monospace;">
                  <strong>Real-life example:</strong><br>
                  "Keep stirring the soup UNTIL it boils"<br>
                  "WHILE there are dirty dishes, wash a dish"
                </p>
              </div>
            </div>

            <!-- Concept 2 -->
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px; border: 2px solid ${config.secondary_action_color || defaultConfig.secondary_action_color};">
              <h2 style="color: ${config.secondary_action_color || defaultConfig.secondary_action_color}; font-size: ${fontSize * 1.75}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                Types of Repetition Structures
              </h2>
              
              <div style="margin-bottom: ${fontSize * 1.5}px;">
                <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.3}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  1. WHILE Loop
                </h3>
                <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize}px; line-height: 1.6; margin-bottom: ${fontSize * 0.75}px;">
                  Checks the condition BEFORE executing the loop body. May never execute if condition is false.
                </p>
                <div style="background: ${config.background_color || defaultConfig.background_color}; padding: ${fontSize * 1.25}px; border-radius: ${fontSize * 0.5}px; font-family: monospace; font-size: ${fontSize * 0.9}px; color: ${config.text_color || defaultConfig.text_color};">
                  SET count TO 1<br>
                  WHILE count ≤ 5 DO<br>
                  &nbsp;&nbsp;OUTPUT count<br>
                  &nbsp;&nbsp;count ← count + 1<br>
                  END WHILE
                </div>
              </div>

              <div>
                <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.3}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  2. REPEAT UNTIL Loop
                </h3>
                <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize}px; line-height: 1.6; margin-bottom: ${fontSize * 0.75}px;">
                  Executes the loop body FIRST, then checks the condition. Always executes at least once.
                </p>
                <div style="background: ${config.background_color || defaultConfig.background_color}; padding: ${fontSize * 1.25}px; border-radius: ${fontSize * 0.5}px; font-family: monospace; font-size: ${fontSize * 0.9}px; color: ${config.text_color || defaultConfig.text_color};">
                  SET count TO 1<br>
                  REPEAT<br>
                  &nbsp;&nbsp;OUTPUT count<br>
                  &nbsp;&nbsp;count ← count + 1<br>
                  UNTIL count > 5
                </div>
              </div>
            </div>

            <!-- Concept 3 -->
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color};">
              <h2 style="color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-size: ${fontSize * 1.75}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                Flowchart Symbols for Repetition
              </h2>
              
              <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(${fontSize * 12}px, 1fr)); gap: ${fontSize * 1.5}px;">
                <div style="text-align: center; padding: ${fontSize}px;">
                  <div style="width: ${fontSize * 6}px; height: ${fontSize * 3}px; background: white; border: 2px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; margin: 0 auto ${fontSize * 0.75}px; border-radius: ${fontSize * 0.25}px;"></div>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 0.9}px; font-weight: bold;">Process Box</p>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 0.75}px;">Actions/statements</p>
                </div>
                
                <div style="text-align: center; padding: ${fontSize}px;">
                  <div style="width: ${fontSize * 6}px; height: ${fontSize * 3}px; background: #fef3c7; border: 2px solid #f59e0b; margin: 0 auto ${fontSize * 0.75}px; transform: rotate(45deg); clip-path: polygon(50% 0%, 100% 50%, 50% 100%, 0% 50%);"></div>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 0.9}px; font-weight: bold; margin-top: ${fontSize * 0.5}px;">Decision Diamond</p>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 0.75}px;">Conditions/loops</p>
                </div>
                
                <div style="text-align: center; padding: ${fontSize}px;">
                  <div style="width: ${fontSize * 6}px; height: ${fontSize * 3}px; background: white; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color}; margin: 0 auto ${fontSize * 0.75}px; border-radius: ${fontSize * 3}px;"></div>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 0.9}px; font-weight: bold;">Oval/Terminal</p>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 0.75}px;">Start/End</p>
                </div>
              </div>
              
              <div style="margin-top: ${fontSize * 1.5}px; padding: ${fontSize * 1.25}px; background: ${config.background_color || defaultConfig.background_color}; border-radius: ${fontSize * 0.5}px;">
                <p style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize}px;">
                  <strong>Key Point:</strong> Arrows that loop back create repetition! The decision diamond controls when to continue or exit the loop.
                </p>
              </div>
            </div>

            <button onclick="navigateTo('practice')" style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%;">
              Continue to Practice →
            </button>
          </div>
        </div>
      `;
    }

    function renderPractice() {
      currentView = 'practice';
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const fontSize = config.font_size || defaultConfig.font_size;
      
      document.getElementById('app').innerHTML = `
        <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px;">
          <div style="max-width: ${fontSize * 62.5}px; margin: 0 auto;">
            <button onclick="navigateTo('home')" style="background: ${config.surface_color || defaultConfig.surface_color}; color: ${config.text_color || defaultConfig.text_color}; border: 1px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 0.75}px ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; margin-bottom: ${fontSize * 2}px; font-size: ${fontSize}px;">
              ← Back to Home
            </button>

            <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2.5}px; font-weight: bold; margin-bottom: ${fontSize * 2}px;">
              ✏️ Practice Flowcharts & Pseudocode
            </h1>

            <!-- Example 1 -->
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px;">
              <h2 style="color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-size: ${fontSize * 1.75}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                Example 1: Sum of Numbers 1 to 10
              </h2>
              
              <div style="display: grid; grid-template-columns: 1fr 1fr; gap: ${fontSize * 2}px;">
                <div>
                  <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.2}px; font-weight: bold; margin-bottom: ${fontSize}px;">Pseudocode:</h3>
                  <div style="background: ${config.background_color || defaultConfig.background_color}; padding: ${fontSize * 1.25}px; border-radius: ${fontSize * 0.5}px; font-family: monospace; font-size: ${fontSize * 0.9}px; color: ${config.text_color || defaultConfig.text_color}; line-height: 1.8;">
                    SET sum TO 0<br>
                    SET count TO 1<br>
                    WHILE count ≤ 10 DO<br>
                    &nbsp;&nbsp;sum ← sum + count<br>
                    &nbsp;&nbsp;count ← count + 1<br>
                    END WHILE<br>
                    OUTPUT sum
                  </div>
                </div>
                
                <div>
                  <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.2}px; font-weight: bold; margin-bottom: ${fontSize}px;">Flowchart:</h3>
                  <svg viewBox="0 0 300 450" style="background: ${config.background_color || defaultConfig.background_color}; border-radius: ${fontSize * 0.5}px; width: 100%;">
                    <defs>
                      <marker id="arrowhead" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
                        <polygon points="0 0, 10 3, 0 6" fill="${config.secondary_action_color || defaultConfig.secondary_action_color}" />
                      </marker>
                    </defs>
                    
                    <ellipse cx="150" cy="30" rx="60" ry="25" class="flowchart-box" />
                    <text x="150" y="37" text-anchor="middle" class="flowchart-text">START</text>
                    
                    <line x1="150" y1="55" x2="150" y2="80" class="flowchart-arrow" />
                    
                    <rect x="90" y="80" width="120" height="40" class="flowchart-box" />
                    <text x="150" y="95" text-anchor="middle" class="flowchart-text">sum = 0</text>
                    <text x="150" y="110" text-anchor="middle" class="flowchart-text">count = 1</text>
                    
                    <line x1="150" y1="120" x2="150" y2="145" class="flowchart-arrow" />
                    
                    <path d="M 150 145 L 220 185 L 150 225 L 80 185 Z" class="flowchart-diamond" />
                    <text x="150" y="185" text-anchor="middle" class="flowchart-text">count ≤ 10?</text>
                    
                    <line x1="220" y1="185" x2="270" y2="185" class="flowchart-arrow" />
                    <text x="245" y="180" text-anchor="middle" class="flowchart-text" fill="${config.text_color || defaultConfig.text_color}">NO</text>
                    
                    <line x1="270" y1="185" x2="270" y2="390" class="flowchart-arrow" />
                    <line x1="270" y1="390" x2="210" y2="390" class="flowchart-arrow" />
                    
                    <line x1="150" y1="225" x2="150" y2="250" class="flowchart-arrow" />
                    <text x="165" y="240" text-anchor="start" class="flowchart-text" fill="${config.text_color || defaultConfig.text_color}">YES</text>
                    
                    <rect x="90" y="250" width="120" height="50" class="flowchart-box" />
                    <text x="150" y="268" text-anchor="middle" class="flowchart-text">sum = sum + count</text>
                    <text x="150" y="288" text-anchor="middle" class="flowchart-text">count = count + 1</text>
                    
                    <line x1="90" y1="275" x2="30" y2="275" class="flowchart-arrow" />
                    <line x1="30" y1="275" x2="30" y2="185" class="flowchart-arrow" />
                    <line x1="30" y1="185" x2="80" y2="185" class="flowchart-arrow" />
                    
                    <rect x="90" y="370" width="120" height="40" class="flowchart-box" />
                    <text x="150" y="395" text-anchor="middle" class="flowchart-text">OUTPUT sum</text>
                    
                    <line x1="150" y1="410" x2="150" y2="430" class="flowchart-arrow" />
                    
                    <ellipse cx="150" cy="445" rx="60" ry="25" class="flowchart-box" />
                    <text x="150" y="452" text-anchor="middle" class="flowchart-text">END</text>
                  </svg>
                </div>
              </div>
            </div>

            <!-- Example 2 -->
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px;">
              <h2 style="color: ${config.secondary_action_color || defaultConfig.secondary_action_color}; font-size: ${fontSize * 1.75}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                Example 2: Password Validation (REPEAT UNTIL)
              </h2>
              
              <div style="display: grid; grid-template-columns: 1fr 1fr; gap: ${fontSize * 2}px;">
                <div>
                  <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.2}px; font-weight: bold; margin-bottom: ${fontSize}px;">Pseudocode:</h3>
                  <div style="background: ${config.background_color || defaultConfig.background_color}; padding: ${fontSize * 1.25}px; border-radius: ${fontSize * 0.5}px; font-family: monospace; font-size: ${fontSize * 0.9}px; color: ${config.text_color || defaultConfig.text_color}; line-height: 1.8;">
                    SET correctPassword TO "1234"<br>
                    REPEAT<br>
                    &nbsp;&nbsp;INPUT userPassword<br>
                    &nbsp;&nbsp;IF userPassword ≠ correctPassword THEN<br>
                    &nbsp;&nbsp;&nbsp;&nbsp;OUTPUT "Wrong! Try again"<br>
                    &nbsp;&nbsp;END IF<br>
                    UNTIL userPassword = correctPassword<br>
                    OUTPUT "Access granted!"
                  </div>
                </div>
                
                <div>
                  <h3 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.2}px; font-weight: bold; margin-bottom: ${fontSize}px;">Flowchart:</h3>
                  <svg viewBox="0 0 300 500" style="background: ${config.background_color || defaultConfig.background_color}; border-radius: ${fontSize * 0.5}px; width: 100%;">
                    <defs>
                      <marker id="arrowhead2" markerWidth="10" markerHeight="10" refX="9" refY="3" orient="auto">
                        <polygon points="0 0, 10 3, 0 6" fill="${config.secondary_action_color || defaultConfig.secondary_action_color}" />
                      </marker>
                    </defs>
                    
                    <ellipse cx="150" cy="30" rx="60" ry="25" class="flowchart-box" />
                    <text x="150" y="37" text-anchor="middle" class="flowchart-text">START</text>
                    
                    <line x1="150" y1="55" x2="150" y2="80" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    
                    <rect x="80" y="80" width="140" height="40" class="flowchart-box" />
                    <text x="150" y="105" text-anchor="middle" class="flowchart-text">correctPassword = "1234"</text>
                    
                    <line x1="150" y1="120" x2="150" y2="145" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    
                    <path d="M 150 145 L 210 185 L 150 225 L 90 185 Z" style="fill: #e0f2fe; stroke: ${config.secondary_action_color || defaultConfig.secondary_action_color}; stroke-width: 2;" />
                    <text x="150" y="190" text-anchor="middle" class="flowchart-text">INPUT</text>
                    <text x="150" y="205" text-anchor="middle" class="flowchart-text">userPassword</text>
                    
                    <line x1="150" y1="225" x2="150" y2="255" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    
                    <path d="M 150 255 L 220 295 L 150 335 L 80 295 Z" class="flowchart-diamond" />
                    <text x="150" y="290" text-anchor="middle" class="flowchart-text">password</text>
                    <text x="150" y="305" text-anchor="middle" class="flowchart-text">correct?</text>
                    
                    <line x1="220" y1="295" x2="270" y2="295" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    <text x="245" y="290" text-anchor="middle" class="flowchart-text" fill="${config.text_color || defaultConfig.text_color}">YES</text>
                    
                    <line x1="270" y1="295" x2="270" y2="435" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    <line x1="270" y1="435" x2="210" y2="435" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    
                    <line x1="150" y1="335" x2="150" y2="365" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    <text x="165" y="352" text-anchor="start" class="flowchart-text" fill="${config.text_color || defaultConfig.text_color}">NO</text>
                    
                    <rect x="90" y="365" width="120" height="40" class="flowchart-box" />
                    <text x="150" y="380" text-anchor="middle" class="flowchart-text">OUTPUT</text>
                    <text x="150" y="395" text-anchor="middle" class="flowchart-text">"Try again"</text>
                    
                    <line x1="90" y1="385" x2="30" y2="385" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    <line x1="30" y1="385" x2="30" y2="185" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    <line x1="30" y1="185" x2="90" y2="185" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    
                    <rect x="90" y="415" width="120" height="40" class="flowchart-box" />
                    <text x="150" y="430" text-anchor="middle" class="flowchart-text">OUTPUT</text>
                    <text x="150" y="445" text-anchor="middle" class="flowchart-text">"Access granted"</text>
                    
                    <line x1="150" y1="455" x2="150" y2="475" class="flowchart-arrow" marker-end="url(#arrowhead2)" />
                    
                    <ellipse cx="150" cy="490" rx="60" ry="25" class="flowchart-box" />
                    <text x="150" y="497" text-anchor="middle" class="flowchart-text">END</text>
                  </svg>
                </div>
              </div>
            </div>

            <button onclick="navigateTo('quiz')" style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%;">
              Ready for the Quiz? →
            </button>
          </div>
        </div>
      `;
    }

    function renderQuiz() {
      currentView = 'quiz';
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const fontSize = config.font_size || defaultConfig.font_size;
      
      if (quizState.currentQuestion === 0 && quizState.studentName === '') {
        document.getElementById('app').innerHTML = `
          <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px; display: flex; align-items: center; justify-content: center;">
            <div style="max-width: ${fontSize * 31.25}px; width: 100%;">
              <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 3}px; text-align: center; box-shadow: 0 10px 40px rgba(0,0,0,0.3);">
                <div style="font-size: ${fontSize * 4}px; margin-bottom: ${fontSize * 1.5}px;">🎮</div>
                <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                  Ready to Test Your Knowledge?
                </h1>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 1.1}px; margin-bottom: ${fontSize * 2}px;">
                  ${quizQuestions.length} questions about repetition structures
                </p>
                
                <label for="studentName" style="display: block; color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px; text-align: left;">
                  Enter your name:
                </label>
                <input 
                  type="text" 
                  id="studentName" 
                  placeholder="Your name here"
                  style="width: 100%; padding: ${fontSize}px; border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color}; border-radius: ${fontSize * 0.5}px; font-size: ${fontSize * 1.1}px; margin-bottom: ${fontSize * 2}px; background: ${config.background_color || defaultConfig.background_color}; color: ${config.text_color || defaultConfig.text_color};"
                />
                
                <button 
                  onclick="startQuiz()"
                  style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.2}px; font-weight: bold; width: 100%;"
                >
                  Start Quiz 🚀
                </button>
                
                <button 
                  onclick="navigateTo('home')"
                  style="background: transparent; color: ${config.text_color || defaultConfig.text_color}; border: 1px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 0.75}px ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize}px; width: 100%; margin-top: ${fontSize}px;"
                >
                  Back to Home
                </button>
              </div>
            </div>
          </div>
        `;
      } else if (quizState.currentQuestion < quizQuestions.length) {
        const question = quizQuestions[quizState.currentQuestion];
        const progress = ((quizState.currentQuestion + 1) / quizQuestions.length) * 100;
        
        document.getElementById('app').innerHTML = `
          <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px;">
            <div style="max-width: ${fontSize * 50}px; margin: 0 auto;">
              <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: ${fontSize * 2}px;">
                <div style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.1}px;">
                  <strong>${quizState.studentName}</strong>
                </div>
                <div style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.1}px;">
                  Question ${quizState.currentQuestion + 1} of ${quizQuestions.length}
                </div>
              </div>
              
              <div style="background: ${config.surface_color || defaultConfig.surface_color}; height: ${fontSize * 0.5}px; border-radius: ${fontSize * 0.25}px; margin-bottom: ${fontSize * 2}px; overflow: hidden;">
                <div style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; height: 100%; width: ${progress}%; transition: width 0.3s;"></div>
              </div>

              <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2.5}px; margin-bottom: ${fontSize * 2}px;">
                <h2 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; font-weight: bold; margin-bottom: ${fontSize * 2}px; line-height: 1.4;">
                  ${question.question}
                </h2>
                
                <div id="options" style="display: grid; gap: ${fontSize}px;">
                  ${question.options.map((option, index) => `
                    <button 
                      onclick="selectAnswer(${index})"
                      class="quiz-option"
                      style="background: ${config.background_color || defaultConfig.background_color}; color: ${config.text_color || defaultConfig.text_color}; border: 2px solid ${config.surface_color || defaultConfig.surface_color}; padding: ${fontSize * 1.25}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; text-align: left; transition: all 0.2s;"
                    >
                      ${option}
                    </button>
                  `).join('')}
                </div>
              </div>
              
              <div id="feedback" style="min-height: ${fontSize * 5}px;"></div>
            </div>
          </div>
        `;
      } else {
        renderQuizComplete();
      }
    }

    function startQuiz() {
      const nameInput = document.getElementById('studentName');
      const name = nameInput.value.trim();
      
      if (name === '') {
        const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
        const fontSize = config.font_size || defaultConfig.font_size;
        nameInput.style.border = `2px solid #ef4444`;
        const errorMsg = document.createElement('p');
        errorMsg.style.color = '#ef4444';
        errorMsg.style.fontSize = `${fontSize * 0.9}px`;
        errorMsg.style.marginTop = `${fontSize * 0.5}px`;
        errorMsg.textContent = 'Please enter your name to continue';
        nameInput.parentNode.insertBefore(errorMsg, nameInput.nextSibling);
        return;
      }
      
      quizState.studentName = name;
      quizState.currentQuestion = 0;
      quizState.score = 0;
      quizState.answers = [];
      renderQuiz();
    }

    function selectAnswer(selectedIndex) {
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontSize = config.font_size || defaultConfig.font_size;
      const question = quizQuestions[quizState.currentQuestion];
      const isCorrect = selectedIndex === question.correct;
      
      quizState.answers.push({
        question: question.question,
        selected: selectedIndex,
        correct: question.correct,
        isCorrect: isCorrect
      });
      
      if (isCorrect) {
        quizState.score++;
      }
      
      const options = document.querySelectorAll('.quiz-option');
      options.forEach((option, index) => {
        option.disabled = true;
        option.style.cursor = 'not-allowed';
        
        if (index === question.correct) {
          option.style.background = '#10b981';
          option.style.color = 'white';
          option.style.borderColor = '#10b981';
        } else if (index === selectedIndex && !isCorrect) {
          option.style.background = '#ef4444';
          option.style.color = 'white';
          option.style.borderColor = '#ef4444';
        } else {
          option.style.opacity = '0.5';
        }
      });
      
      const feedback = document.getElementById('feedback');
      feedback.innerHTML = `
        <div style="background: ${isCorrect ? '#10b981' : '#ef4444'}; color: white; padding: ${fontSize * 1.5}px; border-radius: ${fontSize * 0.75}px; margin-bottom: ${fontSize}px;" class="animate-celebration">
          <div style="font-size: ${fontSize * 2}px; margin-bottom: ${fontSize * 0.5}px;">
            ${isCorrect ? '✓ Correct!' : '✗ Incorrect'}
          </div>
          <p style="font-size: ${fontSize}px; line-height: 1.6;">
            ${question.explanation}
          </p>
        </div>
        <button 
          onclick="nextQuestion()"
          style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%;"
        >
          ${quizState.currentQuestion + 1 < quizQuestions.length ? 'Next Question →' : 'See Results 🎉'}
        </button>
      `;
    }

    function nextQuestion() {
      quizState.currentQuestion++;
      renderQuiz();
    }

    async function renderQuizComplete() {
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const fontSize = config.font_size || defaultConfig.font_size;
      const percentage = Math.round((quizState.score / quizQuestions.length) * 100);
      
      let grade = '';
      let message = '';
      let emoji = '';
      
      if (percentage >= 90) {
        grade = 'A+';
        message = 'Outstanding! You have mastered repetition structures!';
        emoji = '🌟';
      } else if (percentage >= 80) {
        grade = 'A';
        message = 'Excellent work! You understand loops very well!';
        emoji = '🎉';
      } else if (percentage >= 70) {
        grade = 'B';
        message = 'Good job! Keep practicing to improve further!';
        emoji = '👍';
      } else if (percentage >= 60) {
        grade = 'C';
        message = 'Fair attempt. Review the concepts and try again!';
        emoji = '📚';
      } else {
        grade = 'D';
        message = 'Keep learning! Review the lessons and practice more.';
        emoji = '💪';
      }

      const saveButton = document.createElement('button');
      saveButton.id = 'saveScoreBtn';
      saveButton.textContent = 'Save Score to Leaderboard';
      saveButton.style.cssText = `background: ${config.secondary_action_color || defaultConfig.secondary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%; margin-bottom: ${fontSize}px;`;
      
      if (records.length >= 999) {
        document.getElementById('app').innerHTML = `
          <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px; display: flex; align-items: center; justify-content: center;">
            <div style="max-width: ${fontSize * 37.5}px; width: 100%;">
              <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 3}px; text-align: center;">
                <div style="font-size: ${fontSize * 5}px; margin-bottom: ${fontSize * 1.5}px;">${emoji}</div>
                <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2.5}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                  Quiz Complete!
                </h1>
                <div style="background: ${config.background_color || defaultConfig.background_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px;">
                  <div style="font-size: ${fontSize * 4}px; color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                    ${quizState.score} / ${quizQuestions.length}
                  </div>
                  <div style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; margin-bottom: ${fontSize * 0.5}px;">
                    ${percentage}% - Grade: ${grade}
                  </div>
                  <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 1.1}px;">
                    ${message}
                  </p>
                </div>
                
                <div style="background: #fef3c7; border: 2px solid #f59e0b; border-radius: ${fontSize * 0.5}px; padding: ${fontSize * 1.25}px; margin-bottom: ${fontSize * 1.5}px;">
                  <p style="color: #92400e; font-size: ${fontSize}px;">
                    ⚠️ <strong>Leaderboard Full:</strong> Maximum limit of 999 scores reached. Your score cannot be saved at this time.
                  </p>
                </div>
                
                <button 
                  onclick="retakeQuiz()"
                  style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%; margin-bottom: ${fontSize}px;"
                >
                  Try Again 🔄
                </button>
                
                <button 
                  onclick="navigateTo('leaderboard')"
                  style="background: transparent; color: ${config.text_color || defaultConfig.text_color}; border: 2px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%; margin-bottom: ${fontSize}px;"
                >
                  View Leaderboard 🏆
                </button>
                
                <button 
                  onclick="navigateTo('home')"
                  style="background: transparent; color: ${config.text_color || defaultConfig.text_color}; border: 1px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 0.75}px ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize}px; width: 100%;"
                >
                  Back to Home
                </button>
              </div>
            </div>
          </div>
        `;
        return;
      }
      
      document.getElementById('app').innerHTML = `
        <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px; display: flex; align-items: center; justify-content: center;">
          <div style="max-width: ${fontSize * 37.5}px; width: 100%;">
            <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 3}px; text-align: center;">
              <div style="font-size: ${fontSize * 5}px; margin-bottom: ${fontSize * 1.5}px;" class="animate-float">${emoji}</div>
              <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2.5}px; font-weight: bold; margin-bottom: ${fontSize}px;">
                Quiz Complete, ${quizState.studentName}!
              </h1>
              
              <div style="background: ${config.background_color || defaultConfig.background_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 2}px; margin-bottom: ${fontSize * 2}px;">
                <div style="font-size: ${fontSize * 4}px; color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                  ${quizState.score} / ${quizQuestions.length}
                </div>
                <div style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.5}px; margin-bottom: ${fontSize * 0.5}px;">
                  ${percentage}% - Grade: ${grade}
                </div>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 1.1}px;">
                  ${message}
                </p>
              </div>
              
              <div id="saveContainer"></div>
              
              <button 
                onclick="retakeQuiz()"
                style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%; margin-bottom: ${fontSize}px;"
              >
                Try Again 🔄
              </button>
              
              <button 
                onclick="navigateTo('leaderboard')"
                style="background: transparent; color: ${config.text_color || defaultConfig.text_color}; border: 2px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%; margin-bottom: ${fontSize}px;"
              >
                View Leaderboard 🏆
              </button>
              
              <button 
                onclick="navigateTo('home')"
                style="background: transparent; color: ${config.text_color || defaultConfig.text_color}; border: 1px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 0.75}px ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize}px; width: 100%;"
              >
                Back to Home
              </button>
            </div>
          </div>
        </div>
      `;
      
      document.getElementById('saveContainer').appendChild(saveButton);
      
      saveButton.addEventListener('click', async () => {
        if (records.length >= 999) {
          saveButton.textContent = '⚠️ Leaderboard is full (999 scores)';
          saveButton.style.background = '#ef4444';
          saveButton.disabled = true;
          return;
        }
        
        saveButton.textContent = 'Saving...';
        saveButton.disabled = true;
        
        const result = await window.dataSdk.create({
          student_name: quizState.studentName,
          quiz_score: quizState.score,
          completed_at: new Date().toISOString(),
          quiz_type: 'Algorithm Repetition'
        });
        
        if (result.isOk) {
          saveButton.textContent = '✓ Score Saved!';
          saveButton.style.background = '#10b981';
          setTimeout(() => {
            navigateTo('leaderboard');
          }, 1000);
        } else {
          saveButton.textContent = '✗ Failed to save';
          saveButton.style.background = '#ef4444';
          saveButton.disabled = false;
        }
      });
    }

    function retakeQuiz() {
      quizState.currentQuestion = 0;
      quizState.score = 0;
      quizState.answers = [];
      renderQuiz();
    }

    function renderLeaderboard() {
      currentView = 'leaderboard';
      const config = window.elementSdk ? window.elementSdk.config : defaultConfig;
      const fontFamily = config.font_family || defaultConfig.font_family;
      const fontSize = config.font_size || defaultConfig.font_size;
      
      const sortedRecords = [...records].sort((a, b) => {
        if (b.quiz_score !== a.quiz_score) {
          return b.quiz_score - a.quiz_score;
        }
        return new Date(a.completed_at) - new Date(b.completed_at);
      });
      
      const top10 = sortedRecords.slice(0, 10);
      
      document.getElementById('app').innerHTML = `
        <div style="background: ${config.background_color || defaultConfig.background_color}; font-family: ${fontFamily}, sans-serif; min-height: 100%; padding: ${fontSize * 2}px;">
          <div style="max-width: ${fontSize * 50}px; margin: 0 auto;">
            <button onclick="navigateTo('home')" style="background: ${config.surface_color || defaultConfig.surface_color}; color: ${config.text_color || defaultConfig.text_color}; border: 1px solid ${config.secondary_action_color || defaultConfig.secondary_action_color}; padding: ${fontSize * 0.75}px ${fontSize * 1.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; margin-bottom: ${fontSize * 2}px; font-size: ${fontSize}px;">
              ← Back to Home
            </button>

            <div style="text-align: center; margin-bottom: ${fontSize * 2}px;">
              <div style="font-size: ${fontSize * 4}px; margin-bottom: ${fontSize}px;" class="animate-float">🏆</div>
              <h1 style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 2.5}px; font-weight: bold; margin-bottom: ${fontSize * 0.5}px;">
                Top Performers
              </h1>
              <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 1.1}px;">
                ${records.length} total ${records.length === 1 ? 'score' : 'scores'} recorded
              </p>
            </div>

            ${top10.length === 0 ? `
              <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 3}px; text-align: center;">
                <div style="font-size: ${fontSize * 3}px; margin-bottom: ${fontSize}px; opacity: 0.5;">📊</div>
                <p style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.7; font-size: ${fontSize * 1.1}px;">
                  No scores yet. Be the first to take the quiz!
                </p>
              </div>
            ` : `
              <div style="display: grid; gap: ${fontSize}px;">
                ${top10.map((record, index) => {
                  const percentage = Math.round((record.quiz_score / quizQuestions.length) * 100);
                  const date = new Date(record.completed_at);
                  const medal = index === 0 ? '🥇' : index === 1 ? '🥈' : index === 2 ? '🥉' : '';
                  
                  return `
                    <div style="background: ${config.surface_color || defaultConfig.surface_color}; border-radius: ${fontSize * 0.75}px; padding: ${fontSize * 1.5}px; display: flex; align-items: center; gap: ${fontSize * 1.5}px; ${index < 3 ? `border: 2px solid ${config.primary_action_color || defaultConfig.primary_action_color};` : ''}">
                      <div style="font-size: ${fontSize * 2}px; min-width: ${fontSize * 3}px; text-align: center;">
                        ${medal || `#${index + 1}`}
                      </div>
                      <div style="flex: 1;">
                        <div style="color: ${config.text_color || defaultConfig.text_color}; font-size: ${fontSize * 1.2}px; font-weight: bold; margin-bottom: ${fontSize * 0.25}px;">
                          ${record.student_name}
                        </div>
                        <div style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.7; font-size: ${fontSize * 0.85}px;">
                          ${date.toLocaleDateString()} at ${date.toLocaleTimeString()}
                        </div>
                      </div>
                      <div style="text-align: right;">
                        <div style="color: ${config.primary_action_color || defaultConfig.primary_action_color}; font-size: ${fontSize * 1.5}px; font-weight: bold;">
                          ${record.quiz_score}/${quizQuestions.length}
                        </div>
                        <div style="color: ${config.text_color || defaultConfig.text_color}; opacity: 0.8; font-size: ${fontSize * 0.9}px;">
                          ${percentage}%
                        </div>
                      </div>
                    </div>
                  `;
                }).join('')}
              </div>
            `}

            <button onclick="navigateTo('quiz')" style="background: ${config.primary_action_color || defaultConfig.primary_action_color}; color: white; border: none; padding: ${fontSize * 1.25}px ${fontSize * 2.5}px; border-radius: ${fontSize * 0.5}px; cursor: pointer; font-size: ${fontSize * 1.1}px; font-weight: bold; width: 100%; margin-top: ${fontSize * 2}px;">
              Take the Quiz 🎮
            </button>
          </div>
        </div>
      `;
    }

    function navigateTo(view) {
      currentView = view;
      renderCurrentView();
    }

    window.navigateTo = navigateTo;
    window.startQuiz = startQuiz;
    window.selectAnswer = selectAnswer;
    window.nextQuestion = nextQuestion;
    window.retakeQuiz = retakeQuiz;

    initializeApp();
  </script>
 <script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'9ae5e996e6c135be',t:'MTc2NTgwMTQwOS4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>

