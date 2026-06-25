import React, { useState, useRef, useEffect } from 'react';
import { Send, Bot } from 'lucide-react';

export default function App() {
  const [messages, setMessages] = useState([
    { id: 1, text: '哈囉！今天過得怎麼樣？不管是想算數學，還是想找我聊聊都可以喔！', sender: 'bot' }
  ]);
  const [inputValue, setInputValue] = useState('');
  const [isTyping, setIsTyping] = useState(false);
  const messagesEndRef = useRef(null);

  // 自動捲動到最新訊息
  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  };

  useEffect(() => {
    scrollToBottom();
  }, [messages, isTyping]);

  // 中文數字轉換成阿拉伯數字（支援簡單的一到十）
  const convertChineseNums = (str) => {
    const chineseDigits = {
      '零': '0', '一': '1', '二': '2', '兩': '2', '三': '3', 
      '四': '4', '五': '5', '六': '6', '七': '7', '八': '8', '九': '9', '十': '10'
    };
    let result = str;
    Object.keys(chineseDigits).forEach(key => {
      result = result.replace(new RegExp(key, 'g'), chineseDigits[key]);
    });
    return result;
  };

  // 數學計算解析與處理
  const parseAndCalculate = (input) => {
    // 1. 先將中文數字轉為阿拉伯數字
    let clean = convertChineseNums(input);

    // 2. 轉換括號（將中大括號轉為圓括號）與常見數學詞彙
    clean = clean
      .replace(/[\[{]/g, '(')
      .replace(/[\]}]/g, ')')
      .replace(/加/g, '+')
      .replace(/減/g, '-')
      .replace(/乘以|乘/g, '*')
      .replace(/除以|除/g, '/')
      .replace(/÷/g, '/')
      .replace(/×|x|X/g, '*')
      .replace(/＝|=/g, '')
      .replace(/？|\?/g, ''); // 移除問號

    // 3. 自動補上被省略的乘號
    // 數字( 轉為 數字*(  例如: 9(542) -> 9*(542)
    clean = clean.replace(/(\d)\(/g, '$1*(');
    // )數字 轉為 )*數字  例如: (5)2 -> (5)*2
    clean = clean.replace(/\)(\d)/g, ')*$1');
    // )( 轉為 )*(      例如: (2)(3) -> (2)*(3)
    clean = clean.replace(/\)\(/g, ')*(');

    // 4. 只保留數字、運算子、小數點和圓括號
    const expression = clean.replace(/[^0-9+\-*/().]/g, '');

    // 5. 確保算式中同時包含數字與至少一個運算符，避免使用者只打單純數字（如「100」）也被拿去計算
    const hasOperator = /[+\-*/]/.test(expression);
    const hasDigit = /[0-9]/.test(expression);

    if (hasOperator && hasDigit) {
      try {
        // 由於前面已經經過嚴格篩選，只留下安全的數學符號，因此執行運算是絕對安全的
        const result = Function(`"use strict"; return (${expression})`)();
        
        if (result !== null && !isNaN(result) && isFinite(result)) {
          // 格式化輸出，避免浮點數精度問題 (例如 0.1 + 0.2)
          return Number(result.toFixed(10)).toString();
        }
      } catch (e) {
        return null;
      }
    }
    return null;
  };

  const handleSend = (e) => {
    e.preventDefault();
    if (!inputValue.trim()) return;

    const userText = inputValue.trim();

    // 加入使用者的訊息
    const newUserMsg = { id: Date.now(), text: userText, sender: 'user' };
    setMessages((prev) => [...prev, newUserMsg]);
    setInputValue('');
    setIsTyping(true);

    // 模擬機器人思考時間 (0.4秒到1秒之間隨機)
    const delay = Math.floor(Math.random() * 600) + 400;
    
    setTimeout(() => {
      // 判斷是否為數學問題
      const mathResult = parseAndCalculate(userText);
      let replyText = '還行';

      if (mathResult !== null) {
        replyText = `答案是：${mathResult}`;
      }

      const newBotMsg = { id: Date.now() + 1, text: replyText, sender: 'bot' };
      setMessages((prev) => [...prev, newBotMsg]);
      setIsTyping(false);
    }, delay);
  };

  return (
    <div className="min-h-screen bg-gray-100 flex items-center justify-center p-4 font-sans">
      {/* 模擬手機螢幕的容器 */}
      <div className="w-full max-w-md bg-white rounded-2xl shadow-xl overflow-hidden flex flex-col h-[80vh]">
        
        {/* 聊天室頭部 */}
        <div className="bg-indigo-600 text-white p-4 flex items-center shadow-md z-10">
          <div className="w-10 h-10 bg-white/20 rounded-full flex items-center justify-center mr-3">
            <Bot size={24} className="text-white" />
          </div>
          <div>
            <h1 className="font-bold text-lg">還行機器人</h1>
          </div>
        </div>

        {/* 訊息顯示區 */}
        <div className="flex-1 overflow-y-auto p-4 space-y-4 bg-slate-50">
          {messages.map((msg) => (
            <div
              key={msg.id}
              className={`flex ${msg.sender === 'user' ? 'justify-end' : 'justify-start'}`}
            >
              <div
                className={`max-w-[75%] px-4 py-2.5 rounded-2xl ${
                  msg.sender === 'user'
                    ? 'bg-indigo-600 text-white rounded-tr-none shadow-sm'
                    : 'bg-white border border-gray-200 text-gray-800 rounded-tl-none shadow-sm'
                }`}
              >
                <p className="whitespace-pre-wrap break-words">{msg.text}</p>
              </div>
            </div>
          ))}
          
          {/* 輸入中動畫 */}
          {isTyping && (
            <div className="flex justify-start">
              <div className="bg-white border border-gray-200 text-gray-500 px-4 py-3 rounded-2xl rounded-tl-none shadow-sm flex items-center space-x-1">
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '0ms' }}></div>
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '150ms' }}></div>
                <div className="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style={{ animationDelay: '300ms' }}></div>
              </div>
            </div>
          )}
          <div ref={messagesEndRef} />
        </div>

        {/* 輸入區 */}
        <div className="p-3 bg-white border-t border-gray-200">
          <form onSubmit={handleSend} className="flex items-center gap-2">
            <input
              type="text"
              value={inputValue}
              onChange={(e) => setInputValue(e.target.value)}
              placeholder="試著輸入「[9(542)+54-3]/9」..."
              className="flex-1 bg-gray-100 text-gray-800 rounded-full px-4 py-3 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:bg-white transition-colors"
            />
            <button
              type="submit"
              disabled={!inputValue.trim() || isTyping}
              className={`p-3 rounded-full flex items-center justify-center transition-colors ${
                !inputValue.trim() || isTyping
                  ? 'bg-gray-200 text-gray-400 cursor-not-allowed'
                  : 'bg-indigo-600 text-white hover:bg-indigo-700 shadow-md hover:shadow-lg'
              }`}
            >
              <Send size={20} className="ml-1" />
            </button>
          </form>
        </div>
        
      </div>
    </div>
  );
}
