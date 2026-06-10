# AE
import tkinter as tk
import webbrowser as wb
from tkinter import colorchooser as cc
from PIL import Image, ImageTk

win = tk.Tk()
win.title("ArtEditor")
win.geometry("500x500")

entry1 = tk.Entry(win, width=40)
entry1.pack(pady=5)
entry2 = tk.Entry(win, width=40)
entry2.pack(pady=5)

slides = []
current_slide = -1

def run_command():
    global current_slide
    cmd = entry1.get().strip()
    arg = entry2.get().strip()
    
    if cmd == "<slide>":
        slides.append({"title": arg, "elements": []})
        current_slide = len(slides) - 1
        lab = tk.Label(win, text=f"📄 Слайд: {arg}", fg="green")
        lab.pack(pady=5)
    
    elif cmd and current_slide >= 0:
        slides[current_slide]["elements"].append((cmd, arg))
        
        if cmd == "</button/>":
            color = cc.askcolor()[1]
            def open_url():
                wb.open(arg)
            btn = tk.Button(win, text="url", command=open_url, bg=color)
            btn.pack(pady=5)
        
        elif cmd == "<c/t/>":
            color = cc.askcolor()[1]
            lab = tk.Label(win, text=arg, fg=color)
            lab.pack(pady=10)
        
        elif cmd == "<t>":
            lab = tk.Label(win, text=arg)
            lab.pack(pady=10)
        
        elif cmd == "<b>":
            lab = tk.Label(win, text=arg, font=("Arial", 10, "bold"))
            lab.pack(pady=10)
        
        elif cmd == "<h1>":
            lab = tk.Label(win, text=arg, font=("Arial", 20, "bold"))
            lab.pack(pady=10)
        
        elif cmd == "<img>":
            try:
                img = Image.open(arg)
                img.thumbnail((200, 200))
                photo = ImageTk.PhotoImage(img)
                label_img = tk.Label(win, image=photo)
                label_img.image = photo
                label_img.pack(pady=10)
            except:
                lab = tk.Label(win, text=f"Ошибка: {arg}", fg="red")
                lab.pack(pady=10)
        
        elif cmd == "<box>":
            frame = tk.Frame(win, relief=tk.RAISED, bd=2, bg="gray")
            frame.pack(pady=10, padx=10, fill=tk.X)
            lab = tk.Label(frame, text=arg, bg="gray")
            lab.pack(padx=10, pady=10)
        
        elif cmd == "<hr>":
            line = tk.Frame(win, height=2, bg="black")
            line.pack(fill=tk.X, pady=10)
        
        elif cmd == "<clear>":
            for widget in win.winfo_children():
                widget.destroy()
            tk.Entry(win, width=40).pack(pady=5)
            tk.Entry(win, width=40).pack(pady=5)
            tk.Button(win, text="ОК", command=run_command).pack(pady=10)
            slides.clear()
            current_slide = -1
            print("Очищено")
        
        elif cmd == "<save>":
            filename = arg if arg else "presentation"
            
            html = '''<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>Презентация ArtEditor</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 0;
            background: #333;
        }
        .slide {
            display: none;
            width: 800px;
            height: 500px;
            background: white;
            margin: 50px auto;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 0 20px rgba(0,0,0,0.5);
            position: relative;
        }
        .slide.active {
            display: block;
        }
        h1 { color: #333; margin-top: 0; }
        button {
            background: #4caf50;
            color: white;
            border: none;
            padding: 10px 20px;
            margin: 5px;
            cursor: pointer;
            border-radius: 5px;
        }
        button:hover { background: #45a049; }
        .box {
            border: 2px solid #888;
            padding: 15px;
            margin: 10px 0;
            border-radius: 8px;
            background: #f9f9f9;
        }
        hr { border: 2px solid #4caf50; margin: 20px 0; }
        .nav {
            text-align: center;
            margin-top: 20px;
        }
        .nav button {
            background: #333;
            margin: 0 10px;
        }
        .counter {
            text-align: center;
            color: white;
            margin-top: 10px;
        }
    </style>
</head>
<body>
'''
            
            for i, slide in enumerate(slides):
                active = "active" if i == 0 else ""
                html += f'<div class="slide {active}" id="slide{i}">'
                html += f'<h1>{slide["title"]}</h1>'
                
                for cmd, arg in slide["elements"]:
                    if cmd == "</button/>":
                        html += f'<button onclick="window.open(\'{arg}\')">{arg[:30]}</button><br>\n'
                    elif cmd == "<t>":
                        html += f'<p>{arg}</p>\n'
                    elif cmd == "<b>":
                        html += f'<p><strong>{arg}</strong></p>\n'
                    elif cmd == "<h1>":
                        html += f'<h1>{arg}</h1>\n'
                    elif cmd == "<box>":
                        html += f'<div class="box">{arg}</div>\n'
                    elif cmd == "<hr>":
                        html += '<hr>\n'
                
                html += '</div>\n'
            
            html += f'''
    <div class="nav">
        <button onclick="prevSlide()">◀ Назад</button>
        <button onclick="nextSlide()">Вперёд ▶</button>
    </div>
    <div class="counter" id="counter">Слайд 1 из {len(slides)}</div>
    <script>
        let currentSlide = 0;
        const totalSlides = {len(slides)};
        
        function showSlide(n) {{
            document.querySelectorAll('.slide').forEach(s => s.classList.remove('active'));
            document.getElementById('slide' + n).classList.add('active');
            document.getElementById('counter').innerText = `Слайд ${{n+1}} из ${{totalSlides}}`;
        }}
        
        function nextSlide() {{
            if (currentSlide + 1 < totalSlides) {{
                currentSlide++;
                showSlide(currentSlide);
            }}
        }}
        
        function prevSlide() {{
            if (currentSlide - 1 >= 0) {{
                currentSlide--;
                showSlide(currentSlide);
            }}
        }}
        
        document.addEventListener('keydown', (e) => {{
            if (e.key === 'ArrowRight') nextSlide();
            if (e.key === 'ArrowLeft') prevSlide();
        }});
    </script>
</body>
</html>'''
            
            with open(f"{filename}.html", "w", encoding="utf-8") as f:
                f.write(html)
            print(f"✅ Презентация сохранена в {filename}.html")
        
        else:
            lab = tk.Label(win, text=f"Неизвестно: {cmd}", fg="orange")
            lab.pack(pady=5)
    
    entry1.delete(0, tk.END)
    entry2.delete(0, tk.END)

tk.Button(win, text="ОК", command=run_command).pack(pady=10)

win.mainloop()