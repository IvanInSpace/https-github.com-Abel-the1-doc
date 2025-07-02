---
order: 4
title: Share
---

Чтобы Claude (Anthropic) мог использовать **MCP (Multi-Command Prompt)** и **создавать репозитории в твоём Git**, нужно выполнить два этапа:



---





## **🔧 Часть 1. Как установить и использовать MCP в Claude**







### **Что такое MCP?**





MCP -- это мультикомандный промпт, который позволяет Claude выполнять **несколько действий** по одной инструкции: например, писать код, сохранять файлы, делать запросы в API и т.д.



**Важно**: на момент середины 2025 года официальной поддержки “MCP” как отдельной функции нет. Под этим термином часто понимается:



-  Разработка кастомной обвязки вокруг Claude API (или Anthropic Messages API)

-  Использование Claude в среде, где можно выполнять цепочки команд (например, в связке с LangChain, Flowise, LlamaIndex, AutoGen и т.д.)







### **Как «установить MCP»?**







#### **Шаг 1. Получи доступ к Claude API**





1. Зарегистрируйся на <https://console.anthropic.com>

2. Создай API key (он будет вида sk-ant-...)







#### **Шаг 2. Настрой окружение**





Установи зависимости:

```
pip install anthropic langchain openai gitpython
```

Создай Python-файл mcp\_[runner.py](http://runner.py) -- это будет движок, который позволяет запускать последовательные команды через Claude:

```
from anthropic import Anthropic
import subprocess, os

client = Anthropic(api_key=os.getenv("CLAUDE_API_KEY"))

def ask_claude(prompt):
    message = client.messages.create(
        model="claude-3-opus-20240229",
        max_tokens=1024,
        messages=[{"role": "user", "content": prompt}]
    )
    return message.content[0].text

def run_command(command):
    result = subprocess.run(command, shell=True, stdout=subprocess.PIPE, stderr=subprocess.PIPE, text=True)
    return result.stdout + "\n" + result.stderr

prompt = "Создай bash-команду для клонирования репозитория и создания файла README.md"

response = ask_claude(prompt)
print("Claude предложил:\n", response)

output = run_command(response.strip())
print("Результат выполнения:\n", output)
```

Таким образом, Claude будет:



-  генерировать команду,

-  запускать её через subprocess,

-  возвращать результат.





**Важно**: это **опасный код**, он должен работать в безопасной среде (sandbox/docker), иначе Claude сможет удалить файлы на твоей машине.



---





## **🚀 Часть 2. Как позволить Claude создавать репозитории в GitHub**







### **Вариант A. Через GitHub CLI и токен**







#### **Шаг 1. Создай GitHub Personal Access Token**





1. Перейди на: <https://github.com/settings/tokens>

2. Создай **Fine-grained token** с доступом:

   

   -  repo (полный доступ к репозиториям)

   -  workflow (если нужно запускать CI/CD)

   





Сохрани токен -- его нужно будет передать в Claude (или использовать в коде).





#### **Шаг 2. Настрой GitHub CLI**





Установи gh:

```
brew install gh      # для macOS
sudo apt install gh  # для Ubuntu/Debian
```

Авторизуйся:

```
gh auth login --with-token < token.txt
```

Где token.txt -- файл с твоим GitHub токеном.





#### **Шаг 3. Команда создания репозитория**





Ты можешь попросить Claude сгенерировать и выполнить команду:

```
gh repo create my-new-repo --public --confirm
```

И после этого -- сгенерировать [README.md](http://README.md), код, пуш:

```
echo "# My New Repo" > README.md
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/your_user/my-new-repo.git
git push -u origin main
```

Ты можешь завернуть эти шаги в один промпт.



---





## **🎯 Что ты можешь автоматизировать?**





Claude может:



-  Генерировать код

-  Создавать и инициализировать локальный git-репозиторий

-  Использовать gh для создания репо в GitHub

-  Пушить код в GitHub

-  Создавать .gitignore, [README.md](http://README.md), LICENSE и т.д.





---





## **🧠 Продвинутый вариант -- через LangChain**





Ты можешь создать **агента** с Claude, который будет:



-  понимать твои задачи

-  создавать репозиторий

-  коммитить изменения

-  пушить в GitHub





Если нужно -- сделаю готовый YAML-файл, скрипт под FastAPI, или dockerized-среду.



---



Хочешь, чтобы я сделал прототип агента на Python, который:



-  читает промпт

-  создаёт код

-  создаёт репо в GitHub

-  пушит всё туда?





Или тебе нужен вариант для работы **прямо в Claude через UI** (например, в интерфейсе Poe или через API в Notion)?