import cv2
import numpy as np
import math
import os

def generate_geom_ascii(image_path, width=38, contrast=1.4, clip_limit=2.0, tile_grid_size=(4, 4)):
    img = cv2.imread(image_path)
    if img is None:
        return None
        
    h_orig, w_orig, _ = img.shape
    # Crop box dimensions: head & shoulders centered
    y1, y2 = 80, 1280
    x1, x2 = 120, 903
    cropped = img[y1:y2, x1:x2]
    
    h_crop, w_crop, _ = cropped.shape
    
    # Grayscale
    gray = cv2.cvtColor(cropped, cv2.COLOR_BGR2GRAY)
    
    # Resize with aspect ratio correction
    aspect_ratio = h_crop / w_crop
    height = int(width * aspect_ratio * 0.52)
    resized_gray = cv2.resize(gray, (width, height), interpolation=cv2.INTER_AREA)
    
    # Custom geometric mask to keep only person (no background frame/plant)
    mask = np.zeros((height, width), dtype=np.uint8)
    cx = width / 2.0
    head_cy = height * 0.32
    head_rx = width * 0.28
    head_ry = height * 0.23
    
    for r in range(height):
        for c in range(width):
            dx = c - cx
            dy = r - head_cy
            if (dx**2 / head_rx**2) + (dy**2 / head_ry**2) <= 1.0:
                mask[r, c] = 1
            if r >= head_cy:
                factor = (r - head_cy) / (height - head_cy)
                sh_w = head_rx + (width * 0.46 - head_rx) * factor
                if abs(c - cx) <= sh_w:
                    mask[r, c] = 1
                    
    # Apply CLAHE
    clahe = cv2.createCLAHE(clipLimit=clip_limit, tileGridSize=tile_grid_size)
    equalized = clahe.apply(resized_gray)
    
    # Apply mask
    equalized_masked = np.where(mask == 1, equalized, 0)
    
    # Characters mapping
    palette = " .:-=+*#%@"
    num_chars = len(palette)
    
    ascii_rows = []
    for r in range(height):
        row = ""
        for c in range(width):
            if mask[r, c] == 0:
                row += " "
            else:
                val = equalized_masked[r, c]
                idx = int(val / 256.0 * num_chars)
                idx = min(idx, num_chars - 1)
                char = palette[idx]
                if char == ' ' and val > 10:
                    char = '.'
                row += char
        ascii_rows.append(row)
        
    return ascii_rows

def build_profile():
    img_path = r"C:\Users\VAJAYA\Desktop\image.jpeg"
    ascii_lines = generate_geom_ascii(img_path)
    if ascii_lines is None:
        print("Failed to generate ASCII art")
        return
        
    # Formatting terminal info
    ORANGE = "\u001b[38;5;208m"
    BLUE = "\u001b[38;5;75m"
    GREY = "\u001b[90m"
    RESET = "\u001b[0m"
    BOLD = "\u001b[1m"
    GREEN = "\u001b[32m"
    
    right_info = [
        ("title", f"{GREEN}{BOLD}sakavssprasanna-jpg{RESET}{GREY}@{RESET}{GREEN}{BOLD}mirai{RESET}"),
        ("separator", f"{GREY}------------------------------------------------------------{RESET}"),
        ("Username", "sakavssprasanna-jpg"),
        ("Name", "Saka Veera Satya Sai Prasanna"),
        ("OS", "Windows 11"),
        ("Kernel", "MirAI B.Tech Student"),
        ("Host", "Pragati Engineering College"),
        ("Shell", "PowerShell / Bash"),
        ("IDE", "Visual Studio Code"),
        ("Education", "B.Tech CSE (Artificial Intelligence & Machine Learning)"),
        ("College", "Pragati Engineering College"),
        ("CGPA", "8.95 / 10"),
        ("Languages", "Python, Java, C, C++, JavaScript, SQL, HTML, CSS"),
        ("Frameworks", "React, Streamlit"),
        ("AI Libraries", "PyTorch, TensorFlow, OpenCV"),
        ("Database", "PostgreSQL, Supabase"),
        ("Cloud", "AWS"),
        ("Version Control", "Git / GitHub"),
        ("Current Focus", "Machine Learning, Deep Learning, Generative AI, Computer Vision, Research"),
        ("Research Interests", "Deep Learning, Generative AI, Computer Vision"),
        ("Achievements", "Amazon Machine Learning Summer School 2026"),
        ("empty", "Research Publication (MAT Journals)"),
        ("empty", "MirAI AI Builder Internship"),
        ("empty", "AICTE Virtual Internships"),
        ("Hackathons", "HackMelaa | MindSprint"),
        ("Email", "sakavssprasanna@gmail.com"),
        ("GitHub", "https://github.com/sakavssprasanna-jpg"),
        ("LinkedIn", "https://www.linkedin.com/in/saka-veera-satya-sai-prasanna-a1a99b32a"),
        ("LeetCode", "https://leetcode.com/u/prasanna_saka/"),
        ("HackerRank", "https://www.hackerrank.com/profile/sakavssprasanna"),
    ]
    
    combined_lines = []
    max_lines = max(len(ascii_lines), len(right_info))
    ascii_w = 38
    
    for i in range(max_lines):
        # Left side
        if i < len(ascii_lines):
            left = ascii_lines[i]
            left = left + " " * (ascii_w - len(left))
        else:
            left = " " * ascii_w
            
        sep = "   "
        
        # Right side
        if i < len(right_info):
            key, val = right_info[i]
            if key == "title" or key == "separator":
                right = val
            elif key == "empty":
                # Align with dots for achievements list
                # Label is blank, so we pad it
                dots_part = f"{GREY}....................{RESET}"
                right = f"{dots_part} {BLUE}{val}{RESET}"
            else:
                # We align using dots as requested
                # E.g. OS.................Windows 11
                # The total space for label + dots is 20 chars
                dots_count = 20 - len(key)
                dots_part = f"{GREY}{'.' * dots_count}{RESET}"
                right = f"{ORANGE}{key}{RESET}{dots_part} {BLUE}{val}{RESET}"
        else:
            right = ""
            
        left_colored = f"{BOLD}{left}{RESET}"
        combined_lines.append(f"{left_colored}{sep}{right}")
        
    terminal_ansi = "\n".join(combined_lines)
    
    # Now assemble the README content
    readme_html = f"""<p align="center">
  <img src="https://capsule-render.vercel.app/api?type=waving&color=0:ff8c00,100:1e90ff&height=220&section=header&text=Saka%20Veera%20Satya%20Sai%20Prasanna&fontSize=36&animation=twinkling&fontColor=ffffff" alt="Header Banner" width="100%" />
</p>

<p align="center">
  <a href="https://git.io/typing-svg">
    <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=22&duration=3000&pause=1000&color=1E90FF&vCenter=true&width=600&lines=AI+Engineer;Machine+Learning+Developer;Computer+Vision+Enthusiast;Artificial+Intelligence+Researcher" alt="Typing SVG" />
  </a>
</p>

<p align="center">
  <a href="https://github.com/sakavssprasanna-jpg" target="_blank">
    <img src="https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white" alt="GitHub" />
  </a>
  <a href="https://www.linkedin.com/in/saka-veera-satya-sai-prasanna-a1a99b32a" target="_blank">
    <img src="https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white" alt="LinkedIn" />
  </a>
  <a href="https://leetcode.com/u/prasanna_saka/" target="_blank">
    <img src="https://img.shields.io/badge/LeetCode-FFA116?style=for-the-badge&logo=leetcode&logoColor=black" alt="LeetCode" />
  </a>
  <a href="https://www.hackerrank.com/profile/sakavssprasanna" target="_blank">
    <img src="https://img.shields.io/badge/HackerRank-2EC866?style=for-the-badge&logo=hackerrank&logoColor=white" alt="HackerRank" />
  </a>
  <a href="mailto:sakavssprasanna@gmail.com" target="_blank">
    <img src="https://img.shields.io/badge/Email-D14836?style=for-the-badge&logo=gmail&logoColor=white" alt="Email" />
  </a>
  <img src="https://komarev.com/normal-badge/?username=sakavssprasanna-jpg&color=1e90ff&style=flat-square&label=PROFILE+VIEWS" alt="Profile Views" />
</p>

## 📝 Career Summary
As an aspiring AI Engineer and Machine Learning Developer, I specialize in building full-stack intelligent systems and computer vision pipelines. With a strong foundation in Deep Learning, Generative AI, and computer science principles, I focus on bridging the gap between research and production-grade applications. Passionate about open-source contribution and continuous learning, I leverage advanced AI frameworks to engineer robust, scalable solutions for real-world problems.

## 💻 Terminal System Profile

```ansi
{terminal_ansi}
```

## 🛠️ Tech Stack

<p align="center">
  <img src="https://skillicons.dev/icons?i=py,java,c,cpp,js,html,css,react,streamlit,pytorch,tensorflow,opencv,postgres,supabase,aws,git,github,vscode,docker&perline=10" alt="My Skills" />
</p>

## 📂 Featured Projects

| 🤖 **AI Code Tutor** | 💼 **AI Career Mentor** |
| :--- | :--- |
| An intelligent companion that analyzes code in real-time, provides contextual feedback, and explains complex algorithms step-by-step. | Generates resume reviews, tailored cover letters, and mock interview practice based on specific job descriptions and user profiles. |
| 🛠️ `Python` `PyTorch` `DeepSeek` `Streamlit` | 🛠️ `React` `JavaScript` `PostgreSQL` `Supabase` |

| 🚦 **Smart Traffic Detection** | 🛡️ **SAHAYA Women's Safety** |
| :--- | :--- |
| Real-time computer vision application that detects vehicles, counts traffic density, and dynamically adjusts traffic light timings. | A comprehensive platform featuring instant SOS alerts, real-time location tracking, and audio/video recording for personal safety. |
| 🛠️ `Python` `OpenCV` `React` `Supabase` | 🛠️ `React` `AI` `Cloud` |

## 🏆 Achievements

- 🎓 **Amazon Machine Learning Summer School 2026**
  - Selected for the prestigious summer school, learning advanced Machine Learning and Deep Learning concepts directly from Amazon scientists.
- 📝 **Research Publication (MAT Journals)**
  - Published a research paper focusing on AI/ML applications in real-world problem-solving.
- 💻 **MirAI AI Builder Internship**
  - Developed and deployed production-grade AI features and interactive web apps.
- 🎓 **AICTE Virtual Internships**
  - Completed government-approved internships focusing on machine learning and database systems.
- 🥇 **National Hackathons**
  - Participated and placed in **HackMelaa** and **MindSprint** hackathons, building prototype solutions in intense sprints.
- 📜 **NPTEL Certifications**
  - Completed various advanced computer science courses with certifications.

## 📜 Professional Certifications

<p align="center">
  <img src="https://img.shields.io/badge/Amazon_ML-FF9900?style=flat-square&logo=amazon&logoColor=white" alt="Amazon" />
  <img src="https://img.shields.io/badge/Meta-044AF4?style=flat-square&logo=meta&logoColor=white" alt="Meta" />
  <img src="https://img.shields.io/badge/ServiceNow-293E40?style=flat-square&logo=servicenow&logoColor=white" alt="ServiceNow" />
  <img src="https://img.shields.io/badge/NPTEL-FF9900?style=flat-square&logoColor=white" alt="NPTEL" />
  <img src="https://img.shields.io/badge/AICTE-FFA500?style=flat-square&logoColor=white" alt="AICTE" />
  <img src="https://img.shields.io/badge/Kaggle-20BEFF?style=flat-square&logo=kaggle&logoColor=white" alt="Kaggle" />
  <img src="https://img.shields.io/badge/HackerRank-2EC866?style=flat-square&logo=hackerrank&logoColor=white" alt="HackerRank" />
</p>

## 📊 GitHub Analytics

<p align="center">
  <img src="https://github-profile-trophy.vercel.app/?username=sakavssprasanna-jpg&theme=dracula&no-bg=true&no-frame=true" alt="Trophies" width="100%" />
</p>

<p align="center">
  <img src="https://github-readme-stats.vercel.app/api?username=sakavssprasanna-jpg&show_icons=true&title_color=ff8c00&text_color=ffffff&icon_color=1e90ff&bg_color=0d1117&border_color=1e90ff" alt="GitHub Stats" width="48%" />
  <img src="https://github-readme-stats.vercel.app/api/top-langs/?username=sakavssprasanna-jpg&layout=compact&title_color=ff8c00&text_color=ffffff&bg_color=0d1117&border_color=1e90ff" alt="Top Languages" width="48%" />
</p>

<p align="center">
  <img src="https://github-readme-streak-stats.herokuapp.com/?user=sakavssprasanna-jpg&background=0d1117&border=1e90ff&stroke=ff8c00&ring=ff8c00&fire=ff8c00&currStreakNum=1e90ff&currStreakLabel=ff8c00&sideNums=ffffff&sideLabels=ffffff" alt="GitHub Streak" width="100%" />
</p>

<br />

<hr />

<p align="center">
  <sub>🚀 <b>Building Intelligent Systems That Solve Real World Problems</b></sub>
</p>
"""
    
    with open("README.md", "w", encoding="utf-8") as f:
        f.write(readme_html)
    print("README.md written successfully.")

if __name__ == "__main__":
    build_profile()

