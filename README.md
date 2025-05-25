import customtkinter as ctk
from tkinter import ttk, messagebox, filedialog
import pandas as pd
from playwright.sync_api import sync_playwright
import time
import random
import threading
from datetime import datetime
import logging
import os

from time import sleep
class FacebookAutomationApp:
    def __init__(self):
        # تعريف المتغيرات الثابتة
        self.CURRENT_USER = "Blue AutoPoster"
        self.CURRENT_TIME = "2025-04-02 02:26:50"
        
        # إعداد التسجيل
        self.setup_logging()
        
        # إعداد النافذة الرئيسية
        self.setup_window()
        
        # تهيئة المتغيرات
        self.browser = None
        self.page = None
        self.post_text = None  # تغيير من قائمة إلى متغير واحد
        self.is_publishing = False
        self.selected_image = None
        
        # إنشاء واجهة المستخدم
        self.create_main_frames()
        self.create_login_section()
        self.create_post_section()  # تغيير من posts إلى post
        self.create_additional_options()
        self.create_groups_section()
        self.create_control_section()
        self.create_status_bar()

    def setup_logging(self):
        """إعداد نظام التسجيل"""
        log_dir = "logs"
        if not os.path.exists(log_dir):
            os.makedirs(log_dir)
        
        log_file = os.path.join(
            log_dir, 
            f"facebook_automation_{datetime.now().strftime('%Y%m%d_%H%M%S')}.log"
        )
        
        logging.basicConfig(
            filename=log_file,
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            encoding='utf-8'
        )
        
        logging.info(f"بدء تشغيل التطبيق - المستخدم: {self.CURRENT_USER}")
        logging.info(f"وقت البدء: {self.CURRENT_TIME}")

    def setup_window(self):
        """إعداد النافذة الرئيسية"""
        ctk.set_appearance_mode("Light")
        self.window = ctk.CTk()
        self.window.title(f"نشر تلقائي في مجموعات فيسبوك - {self.CURRENT_USER}")
        self.window.geometry("1200x800")
        self.window.minsize(1000, 600)
        
        # تعيين دالة الإغلاق
        self.window.protocol("WM_DELETE_WINDOW", self.on_closing)

    # def change_mode(choice):
    #     ctk.set_appearance_mode(choice)

    # root = ctk.CTk()
    # root.geometry("300x150")

    # option_menu = ctk.CTkOptionMenu(root, values=["Light", "Dark", "System"], command=change_mode)
    # option_menu.set("Dark")  # الوضع الافتراضي
    # option_menu.pack(pady=20)

    # root.mainloop()





    def create_main_frames(self):
        """إنشاء الإطارات الرئيسية"""
        # إطار التحكم (يسار)
        self.control_frame = ctk.CTkFrame(self.window, width=400)
        self.control_frame.grid(row=0, column=0, padx=10, pady=10, sticky="nsew")
        self.control_frame.grid_propagate(False)
        
        # إطار المجموعات (يمين)
        self.groups_frame = ctk.CTkFrame(self.window)
        self.groups_frame.grid(row=0, column=1, padx=10, pady=10, sticky="nsew")
        
        # تكوين الشبكة
        self.window.grid_columnconfigure(1, weight=1)
        self.window.grid_rowconfigure(0, weight=1)

    def create_login_section(self):
        """إنشاء قسم تسجيل الدخول"""
        login_frame = ctk.CTkFrame(self.control_frame)
        login_frame.pack(fill="x", padx=10, pady=10)
        
        # عنوان القسم
        ctk.CTkLabel(
            login_frame, 
            text="بيانات تسجيل الدخول",
            font=("Arial", 14, "bold")
        ).pack(pady=5)
        
        # إطار وضع تسجيل الدخول
        mode_frame = ctk.CTkFrame(login_frame)
        mode_frame.pack(fill="x", pady=5)
        
        # متغير وضع تسجيل الدخول
        self.login_mode_var = ctk.StringVar(value="auto")
        
        # أزرار الاختيار
        ctk.CTkRadioButton(
            mode_frame,
            text="تسجيل دخول تلقائي",
            variable=self.login_mode_var,
            value="auto",
            command=self.toggle_login_mode
        ).pack(side="left", padx=5)
        
        ctk.CTkRadioButton(
            mode_frame,
            text="تسجيل دخول يدوي",
            variable=self.login_mode_var,
            value="manual",
            command=self.toggle_login_mode
        ).pack(side="left", padx=5)
        
        # حقول إدخال البيانات
        self.email_entry = ctk.CTkEntry(
            login_frame, 
            placeholder_text="البريد الإلكتروني"
        )
        self.email_entry.pack(fill="x", padx=5, pady=5)
        
        self.password_entry = ctk.CTkEntry(
            login_frame, 
            placeholder_text="كلمة المرور",
            show="*"
        )
        self.password_entry.pack(fill="x", padx=5, pady=5)

    def toggle_login_mode(self):
        """التبديل بين وضع تسجيل الدخول التلقائي واليدوي"""
        is_auto = self.login_mode_var.get() == "auto"
        self.email_entry.configure(state="normal" if is_auto else "disabled")
        self.password_entry.configure(state="normal" if is_auto else "disabled")











    def create_post_section(self):
        """إنشاء قسم المنشور"""
        posts_frame = ctk.CTkFrame(self.control_frame)
        posts_frame.pack(fill="x", padx=10, pady=10)
        
        # عنوان القسم
        ctk.CTkLabel(
            posts_frame, 
            text="نص المنشور",
            font=("Arial", 14, "bold")
        ).pack(pady=5)
        
        # مربع نص المنشور
        self.post_text = ctk.CTkTextbox(
            posts_frame, 
            height=150,
            font=("Arial", 12)
        )
        self.post_text.pack(fill="x", padx=5, pady=5)

    def create_additional_options(self):
        """إنشاء قسم الخيارات الإضافية"""
        options_frame = ctk.CTkFrame(self.control_frame)
        options_frame.pack(fill="x", padx=10, pady=10)
        
        # عنوان القسم
        ctk.CTkLabel(
            options_frame,
            text="خيارات إضافية",
            font=("Arial", 14, "bold")
        ).pack(pady=5)
        
        # إطار موقع النص الإضافي
        text_position_frame = ctk.CTkFrame(options_frame)
        text_position_frame.pack(fill="x", pady=5)
        
        # متغير تفعيل النص الإضافي
        self.add_text_var = ctk.BooleanVar(value=False)
        self.add_text_check = ctk.CTkCheckBox(
            text_position_frame,
            text="إضافة نص",
            variable=self.add_text_var
        )
        self.add_text_check.pack(side="left", padx=5)
        
        # متغير موقع النص
        self.text_position_var = ctk.StringVar(value="end")
        
        # أزرار اختيار موقع النص
        ctk.CTkRadioButton(
            text_position_frame,
            text="في البداية",
            variable=self.text_position_var,
            value="start"
        ).pack(side="left", padx=5)
        
        ctk.CTkRadioButton(
            text_position_frame,
            text="في النهاية",
            variable=self.text_position_var,
            value="end"
        ).pack(side="left", padx=5)
        
        # إطار نوع النص الإضافي
        text_type_frame = ctk.CTkFrame(options_frame)
        text_type_frame.pack(fill="x", pady=5)
        
        # متغير نوع النص
        self.text_type_var = ctk.StringVar(value="group_name")
        
        # أزرار اختيار نوع النص
        text_types = [
            ("اسم المجموعة", "group_name"),
            ("نص عشوائي", "random_text"),
            ("رقم عشوائي", "random_number")
        ]
        
        for text, value in text_types:
            ctk.CTkRadioButton(
                text_type_frame,
                text=text,
                variable=self.text_type_var,
                value=value
            ).pack(side="left", padx=5)
        
        # إطار إضافة الصور
        image_frame = ctk.CTkFrame(options_frame)
        image_frame.pack(fill="x", pady=5)
        
        # متغير تفعيل إضافة الصور
        self.add_image_var = ctk.BooleanVar(value=False)
        self.add_image_check = ctk.CTkCheckBox(
            image_frame,
            text="إضافة صورة للمنشور",
            variable=self.add_image_var
        )
        self.add_image_check.pack(side="left", padx=5)
        
        # زر اختيار الصورة
        self.select_image_button = ctk.CTkButton(
            image_frame,
            text="اختيار صورة",
            command=self.select_image,
            width=120
        )
        self.select_image_button.pack(side="left", padx=5)
        
        # عنوان الصورة المختارة
        self.image_label = ctk.CTkLabel(
            image_frame, 
            text="لم يتم اختيار صورة"
        )
        self.image_label.pack(side="left", padx=5)

    def select_image(self):
        """دالة اختيار الصورة"""
        file_types = (
            ('صور', '*.jpg;*.jpeg;*.png'),
            ('كل الملفات', '*.*')
        )
        filename = filedialog.askopenfilename(
            title='اختر صورة',
            filetypes=file_types
        )
        
        if filename:
            self.selected_image = filename
            self.image_label.configure(text=os.path.basename(filename))
            logging.info(f"تم اختيار الصورة: {filename}")

    def create_groups_section(self):
        """إنشاء قسم المجموعات"""
        # عنوان القسم
        ctk.CTkLabel(
            self.groups_frame,
            text="قائمة المجموعات",
            font=("Arial", 16, "bold")
        ).pack(pady=5)
        
        # إطار الجدول
        tree_frame = ttk.Frame(self.groups_frame)
        tree_frame.pack(fill="both", expand=True, padx=5, pady=5)













                # تعريف نمط الجدول
        style = ttk.Style()
        style.configure(
            "Custom.Treeview",
            background="#2B2B2B",
            foreground="white",
            fieldbackground="#2B2B2B",
            rowheight=25
        )
        
        # إنشاء الجدول
        self.groups_tree = ttk.Treeview(
            tree_frame,
            columns=("name", "link", "status", "time", "attempts"),
            show="headings",
            style="Custom.Treeview"
        )
        
        # تعيين عناوين الأعمدة
        columns_config = {
            "name": ("اسم المجموعة", 200),
            "link": ("رابط المجموعة", 250),
            "status": ("الحالة", 100),
            "time": ("وقت النشر", 150),
            "attempts": ("المحاولات", 80)
        }
        
        for col, (heading, width) in columns_config.items():
            self.groups_tree.heading(col, text=heading)
            self.groups_tree.column(col, width=width, anchor="center")
        
        # إضافة أشرطة التمرير
        scrollbars = {
            "y": ttk.Scrollbar(tree_frame, orient="vertical"),
            "x": ttk.Scrollbar(tree_frame, orient="horizontal")
        }
        
        scrollbars["y"].config(command=self.groups_tree.yview)
        scrollbars["x"].config(command=self.groups_tree.xview)
        self.groups_tree.config(yscrollcommand=scrollbars["y"].set,
                              xscrollcommand=scrollbars["x"].set)
        
        # ترتيب العناصر
        self.groups_tree.pack(side="left", fill="both", expand=True)
        scrollbars["y"].pack(side="right", fill="y")
        scrollbars["x"].pack(side="bottom", fill="x")

    def create_control_section(self):
        """إنشاء قسم أزرار التحكم"""
        # إنشاء إطار للتحكم
        control_frame = ctk.CTkFrame(self.control_frame)
        control_frame.pack(fill="x", padx=10, pady=10, side="bottom")
        
        # إطار الفاصل الزمني
        time_frame = ctk.CTkFrame(control_frame)
        time_frame.pack(fill="x", pady=(0, 5))
        
        ctk.CTkLabel(
            time_frame,
            text="الفاصل الزمني (ثانية):",
            font=("Arial", 12)
        ).pack(side="left", padx=5)
        
        self.time_interval = ctk.CTkEntry(time_frame, width=70)
        self.time_interval.pack(side="left", padx=5)
        self.time_interval.insert(0, "10")
        
        # إطار الأزرار
        buttons_frame = ctk.CTkFrame(control_frame)
        buttons_frame.pack(fill="x", pady=5)
        
        # زر بدء النشر
        self.start_button = ctk.CTkButton(
            buttons_frame,
            text="بدء النشر",
            command=self.start_publishing,
            fg_color="#28a745",
            hover_color="#218838",
            font=("Arial", 14, "bold")
        )
        self.start_button.pack(side="left", padx=5, fill="x", expand=True)
        
        # زر تحميل المجموعات
        self.load_button = ctk.CTkButton(
            buttons_frame,
            text="تحميل من Excel",
            command=self.load_from_excel,
            fg_color="#17a2b8",
            hover_color="#138496",
            font=("Arial", 14)
        )
        self.load_button.pack(side="left", padx=5, fill="x", expand=True)
        
        # زر إيقاف النشر
        self.stop_button = ctk.CTkButton(
            buttons_frame,
            text="إيقاف النشر",
            command=self.stop_publishing,
            fg_color="#dc3545",
            hover_color="#c82333",
            font=("Arial", 14),
            state="disabled"
        )
        self.stop_button.pack(side="left", padx=5, fill="x", expand=True)

    def create_status_bar(self):
        """إنشاء شريط الحالة"""
        self.status_label = ctk.CTkLabel(
            self.window,
            text="جاهز للنشر",
            anchor="w",
            font=("Arial", 12)
        )
        self.status_label.grid(
            row=1,
            column=0,
            columnspan=2,
            sticky="ew",
            padx=10,
            pady=5
        )

    def update_status_label(self, text):
        """تحديث نص شريط الحالة"""
        try:
            self.status_label.configure(text=text)
            self.window.update_idletasks()
        except:
            pass

    def update_group_status(self, item, status, post_time="-", attempts="0"):
        """تحديث حالة المجموعة في الجدول"""
        try:
            values = list(self.groups_tree.item(item)['values'])
            values[2] = status  # الحالة
            values[3] = post_time  # وقت النشر
            values[4] = attempts  # عدد المحاولات
            self.groups_tree.item(item, values=values)
            self.window.update_idletasks()
        except:
            pass
























    def load_from_excel(self):
        """تحميل المجموعات من ملف Excel"""
        try:
            df = pd.read_excel('data_groups.xlsx', header=None)
            logging.info(f"تم قراءة الملف بنجاح. عدد الصفوف: {len(df)}")
            
            # مسح المجموعات الحالية
            for item in self.groups_tree.get_children():
                self.groups_tree.delete(item)
            
            # تحديد الصفوف الصالحة وإضافتها
            valid_rows = df.iloc[3:].dropna(how='all')
            added_groups = 0
            
            for index, row in valid_rows.iterrows():
                try:
                    group_link = str(row[0]).strip()
                    group_name = str(row[1]).strip()
                    
                    if pd.notna(group_link) and pd.notna(group_name) and \
                       group_link != "nan" and group_name != "nan":
                        self.groups_tree.insert(
                            "", 
                            "end",
                            values=(group_name, group_link, "لم يتم النشر", "-", "0")
                        )
                        added_groups += 1
                        logging.info(f"تمت إضافة المجموعة: {group_name}")
                except Exception as e:
                    logging.error(f"خطأ في إضافة الصف {index}: {str(e)}")
                    continue
            
            if added_groups > 0:
                messagebox.showinfo("نجاح", f"تم تحميل {added_groups} مجموعة بنجاح")
            else:
                messagebox.showwarning("تنبيه", "لم يتم العثور على مجموعات صالحة في الملف")
                
        except FileNotFoundError:
            messagebox.showerror("خطأ", "ملف data_groups.xlsx غير موجود")
            logging.error("ملف Excel غير موجود")
        except Exception as e:
            messagebox.showerror("خطأ", f"حدث خطأ أثناء قراءة الملف: {str(e)}")
            logging.error(f"خطأ في تحميل الملف: {str(e)}")

    def get_additional_text(self, group_name):
        """إنشاء النص الإضافي للمنشور"""
        if not self.add_text_var.get():
            return ""
            
        text_type = self.text_type_var.get()
        
        if text_type == "group_name":
            return f"\n\nمنشور في مجموعة: {group_name}"
        elif text_type == "random_text":
            texts = [
                "#تم_النشر",
                "#منشور_جديد",
                "#شارك_المنشور",
                "#نشر_تلقائي",
                f"#منشور_{datetime.now().strftime('%Y%m%d')}"
            ]
            return f"\n\n{random.choice(texts)}"
        elif text_type == "random_number":
            return f"\n\n#{random.randint(1000, 9999)}"
        return ""

    def validate_inputs(self):
        """التحقق من صحة المدخلات"""
        # التحقق من وجود نص المنشور
        if not self.post_text.get("1.0", "end-1c").strip():
            messagebox.showwarning("تنبيه", "يرجى إدخال نص المنشور")
            return False
            
        # التحقق من وجود مجموعات
        if not self.groups_tree.get_children():
            messagebox.showwarning("تنبيه", "يرجى إضافة مجموعات أولاً")
            return False
            
        # التحقق من بيانات تسجيل الدخول في الوضع التلقائي
        if self.login_mode_var.get() == "auto":
            if not self.email_entry.get() or not self.password_entry.get():
                messagebox.showwarning("تنبيه", "يرجى إدخال بيانات تسجيل الدخول")
                return False
                
        # التحقق من الفاصل الزمني
        try:
            interval = int(self.time_interval.get())
            if interval < -1:
                messagebox.showwarning("تنبيه", "يجب أن يكون الفاصل الزمني 10 ثوانٍ على الأقل")
                return False
        except ValueError:
            messagebox.showwarning("تنبيه", "يرجى إدخال فاصل زمني صحيح")
            return False
            
        # التحقق من الصورة إذا تم تفعيل خيار إضافة الصور
        if self.add_image_var.get() and not hasattr(self, 'selected_image'):
            messagebox.showwarning("تنبيه", "يرجى اختيار صورة أولاً")
            return False
            
        return True
    











    def start_publishing(self):
        """بدء عملية النشر"""
        if not self.validate_inputs():
            return
            
        # تحديث حالة الأزرار
        self.start_button.configure(state="disabled")
        self.load_button.configure(state="disabled")
        self.stop_button.configure(state="normal")
        self.is_publishing = True
        
        # بدء عملية النشر في thread منفصل
        threading.Thread(target=self._publish_process, daemon=True).start()

    def stop_publishing(self):
        """إيقاف عملية النشر"""
        self.is_publishing = False
        self.stop_button.configure(state="disabled")
        self.update_status_label("جاري إيقاف النشر...")
        logging.info("تم إيقاف عملية النشر بواسطة المستخدم")

    def _publish_process(self):
        """عملية النشر الرئيسية"""
        try:
            self.update_status_label("جاري بدء المتصفح...")
            logging.info("بدء عملية النشر")
            
            # with sync_playwright() as playwright:
            #     # إعداد المتصفح
            #     self.browser = playwright.chromium.launch(
            #         headless=False,
            #         args=['--start-maximized']
            #     )
            #     self.page = self.browser.new_page(no_viewport=True)


            # with sync_playwright() as playwright:
            #     # إعداد المتصفح بمجلد بيانات مستمر
            #     self.browser = playwright.chromium.launch_persistent_context(
            #         "my_data",  # مجلد بيانات المستخدم
            #         headless=False,
            #         args=['--start-maximized'],
            #         no_viewport=True
            #     )
            #     # إنشاء صفحة من السياق المستمر
            #     self.page = self.browser.new_page()

            with sync_playwright() as playwright:
                # إعداد المتصفح بمجلد بيانات مستمر
                self.browser = playwright.chromium.launch_persistent_context(
                    "my_data",  # مجلد بيانات المستخدم
                    headless=False,
                    args=['--start-maximized'],
                    no_viewport=True
                )
                # إنشاء صفحة من السياق المستمر
                self.page = self.browser.new_page()


                

                # تسجيل الدخول
                self.update_status_label("جاري تسجيل الدخول...")
                self.page.goto("https://www.facebook.com")

                if self.login_mode_var.get() == "auto":
                    # تسجيل دخول تلقائي
                    self.page.fill('input[name="email"]', self.email_entry.get())
                    self.page.fill('input[name="pass"]', self.password_entry.get())
                    self.page.click('button[name="login"]')
                else:
                    # تسجيل دخول يدوي
                    messagebox.showinfo("تسجيل الدخول", "قم بتسجيل الدخول يدوياً في المتصفح")
                
                # انتظار تسجيل الدخول
                time.sleep(30)
                print("Go post")

               

                max_attempts = 5
                attempts = 0

                while attempts < max_attempts:
                    current_url = self.page.url
                    if current_url == "https://www.facebook.com/":
                        print("تم")
                        break
                    else:
                        # print(f"المحاولة {attempts + 1}: الرابط الحالي هو {current_url} - في الانتظار 10 ثوانٍ...")
                        time.sleep(10)
                        attempts += 1



                
                # البدء في النشر
                interval = int(self.time_interval.get())
                groups = self.groups_tree.get_children()
                total_groups = len(groups)
                current_group = 1
                
                for item in groups:
                    if not self.is_publishing:
                        break
                        
                    group_data = self.groups_tree.item(item)['values']
                    group_name = group_data[0]
                    group_link = group_data[1]
                    
                    self.update_status_label(f"جاري النشر في المجموعة {current_group} من {total_groups}: {group_name}")
                    
                    if self.post_to_group(item, group_data):
                        logging.info(f"تم النشر بنجاح في المجموعة: {group_name}")
                    else:
                        print('no post')
                        # logging.error(f"فشل النشر في المجموعة: {group_name}")
                    
                    current_group += 1
                    
                    if self.is_publishing and current_group <= total_groups:
                        self.update_status_label(f"انتظار {interval} ثانية قبل المجموعة التالية...")
                        time.sleep(interval)
                
                # إنهاء العملية
                self.browser.close()
                
                if self.is_publishing:
                    self.update_status_label("تم الانتهاء من النشر في جميع المجموعات")
                    messagebox.showinfo("اكتمل", "تم الانتهاء من النشر في جميع المجموعات")
                else:
                    self.update_status_label("تم إيقاف النشر")
                    
        except Exception as e:
            error_msg = f"حدث خطأ أثناء النشر: {str(e)}"
            logging.error(error_msg)
            messagebox.showerror("خطأ", error_msg)
            self.update_status_label("حدث خطأ أثناء النشر")
        
        finally:
            self.is_publishing = False
            self.start_button.configure(state="normal")
            self.load_button.configure(state="normal")
            self.stop_button.configure(state="disabled")









    def post_to_group(self, item, group_data ):
        """النشر في مجموعة محددة مع تحسين الاستقرار والتفاعل مع العنصر"""


        



        max_retries = 2
        current_retry = 0

        while current_retry < max_retries and self.is_publishing:
            try:
                # تحديث الحالة
                self.update_group_status(item, "جاري المحاولة", "-", str(current_retry + 1))

                # 1. الانتقال إلى صفحة المجموعة
                try:
                    self.page.goto(group_data[1], timeout=60000)
                    self.page.wait_for_selector('div[role="main"]', timeout=60000)
                    self.page.wait_for_timeout(9000)
                except Exception as e:
                    logging.warning(f"خطأ أثناء تحميل الصفحة: {e}")

                time.sleep(5)

                # 2. تجهيز نص المنشور
                post_content = self.post_text.get("1.0", "end-1c").strip()
                additional_text = self.get_additional_text(group_data[0])
                if self.text_position_var.get() == "start":
                    post_content = additional_text + "\n\n" + post_content
                else:
                    post_content = post_content + additional_text

                # 3. إغلاق النوافذ المنبثقة إن وجدت
                for sel in ["//div[@aria-label='Close']", "//div[@aria-label='إغلاق']", "//div[@aria-label='Not now']"]:
                    try:
                        popup = self.page.locator(sel).first
                        if popup.is_visible(timeout=2000):
                            popup.click()
                            time.sleep(1)
                    except:
                        pass
                print('1')            
                # 4. النقر على زر الكتابة
                clicked = False

                # try:
                #     # استخدم locator دقيق لزر الكتابة
                #     post_button = self.page.locator('span:has-text("اكتب شيئًا...")').first
                #     if post_button.is_visible(timeout=5000):
                #         post_button.click()
                #         clicked = True
                #     else:
                #         post_button_en = self.page.locator('span:has-text("Write something...")').first
                #         if post_button_en.is_visible(timeout=5000):
                #             post_button_en.click()
                #             clicked = True
                # except Exception as e:
                #     logging.debug(f"فشل في النقر على زر الكتابة: {e}")




                try:
                    clicked = False  # تأكد من وجود هذا المتغير لتتبع النقر
                    # المحاولة الأولى: بالعربية "اكتب شيئًا..."
                    post_button = self.page.locator('span:has-text("اكتب شيئًا...")').first
                    if post_button.is_visible(timeout=5000):
                        post_button.click()
                        clicked = True
                    else:
                        # المحاولة الثانية: بالإنجليزية "Write something..."
                        post_button_en = self.page.locator('span:has-text("Write something...")').first
                        if post_button_en.is_visible(timeout=5000):
                            post_button_en.click()
                            clicked = True
                        else:
                            # المحاولة الثالثة: "بدء مناقشة"
                            post_button_discuss = self.page.locator('span:has-text("بدء مناقشة")').first
                            if post_button_discuss.is_visible(timeout=5000):
                                post_button_discuss.click()
                                clicked = True
                except Exception as e:
                    logging.debug(f"فشل في النقر على زر الكتابة: {e}")












                if not clicked:
                    raise Exception("لم يتم العثور على زر كتابة المنشور")
                
                # post_content = "12343456457568789690"
                time.sleep(13)
                print('2')  

                # 5. التفاعل مع مربع النص بدقة أكبر
                try:
                    # استهداف مربع النص باستخدام role="textbox"
                    post_input = self.page.locator('div[role="textbox"][contenteditable="true"]').first
                    print('2-1')
                    # post_input.wait_for(state="visible", timeout=2000)
                    print('2-2')
                    # post_input.fill(post_content)
                    # self.page.fill(post_content)
                    print('3')  
                except Exception:
                    try:
                        # fallback: الكتابة عبر لوحة المفاتيح
                        self.page.keyboard.type(post_content)
                    except Exception as e:
                        raise Exception(f"فشل في كتابة المنشور: {e}")

                print('4')  
                time.sleep(2)






                # # 6. رفع الصورة إن وجدت
                # if self.add_image_var.get() and hasattr(self, 'selected_image'):
                #     for sel in ["//div[@aria-label='صورة/فيديو']", "//div[@aria-label='Photo/video']", "//input[@accept='image/*']"]:
                #         try:
                #             elem = self.page.locator(sel).first
                #             if elem.is_visible(timeout=3000):
                #                 if sel.startswith('//input'):
                #                     elem.set_input_files(self.selected_image)
                #                 else:
                #                     elem.click()
                #                     time.sleep(1)
                #                     self.page.locator("input[type='file']").first.set_input_files(self.selected_image)
                #                 time.sleep(5)
                #                 break
                #         except:
                #             continue



                # # # 6. رفع الصورة إن وجدت
                # if self.add_image_var.get() and hasattr(self, 'selected_image'):
                #     for sel in ["//div[@aria-label='صورة/فيديو']", "//div[@aria-label='Photo/video']", "//input[@accept='image/*']"]:
                #         try:
                            # elem = self.page.locator(sel).first
                            # if elem.is_visible(timeout=3000):
                            #     # if sel.startswith('//input'):
                            #         # elem.set_input_files(self.selected_image)
                            #     else:
                            #         elem.click()
                            #         time.sleep(1)
                            #         self.page.locator("input[type='file']").first.set_input_files(self.selected_image)
                            #     time.sleep(5)
                            #     break
                        #     image_path = self.selected_image
                        # except:
                        #     continue


                # import os
                # import logging
                # import time

                # selected_image = os.path.join(os.getcwd(), "img.png")  #######        ---<<<<<
    



                
                # 6. رفع الصورة تلقائيًا إن وُجدت
                if self.add_image_var.get() and hasattr(self, 'selected_image'):
                    try:
                        print("✅ llll")
                        # تحديد مسار الصورة
                        # image_path = os.path.join(os.getcwd(), "img.png")  #######        ---<<<<<
                        image_path = self.selected_image
                        # write_button = self.page.locator('span:has-text("اكتب شيئًا...")')
                        # write_button.wait_for(timeout=10000)  # انتظار ظهور الزر
                        # write_button.click()
                        time.sleep(11)
                        
                        # 1. تحديد منطقة النشر
                        post_area = self.page.locator('div[aria-placeholder="إنشاء منشور عام..."]')
                        post_area.click()
                        time.sleep(5)
                        # 2. انتظر ظهور FileChooser عند النقر على زر إضافة الصور
                        with self.page.expect_file_chooser() as file_chooser_info:
                            # النقر على زر إضافة الصور
                            self.page.locator('div[aria-label="صورة/فيديو"]').click()
                        
                        # 3. الحصول على file_chooser واختيار الملف
                        file_chooser = file_chooser_info.value
                        file_chooser.set_files(image_path)
                        print("✅ llll")
                    except Exception as e :
                        print(e)
                    # print(e)

                    # مسار الصورة الثابت (عدله حسب مكان وجود الصورة)
                    # image_path = os.path.abspath("img.png")

                    # تحقق أن الملف موجود
                    if not os.path.exists(image_path):
                        logging.error(f"❌ ملف الصورة غير موجود: {image_path}")
                    else:
                        print("no img")

                        
                            
                        # 5. التفاعل مع مربع النص بدقة أكبر
                        try:
                            post_area = self.page.locator('div[aria-placeholder="إنشاء منشور عام..."]')
                            # post_area.click()
                            # استهداف مربع النص باستخدام role="textbox"
                            post_input = self.page.locator('div[role="textbox"][contenteditable="true"]').first
                            print('2-1')
                            post_input.wait_for(state="visible", timeout=5000)
                            time.sleep(9)
                            print('2-2')
                            self.page.fill(post_content)
                            print('3')  
                        except Exception:
                            try:
                                # fallback: الكتابة عبر لوحة المفاتيح
                                self.page.keyboard.type(post_content)
                            except Exception as e:
                                raise Exception(f"فشل في كتابة المنشور: {e}")
                        print('4')  
                        time.sleep(2)
                        print('444')  
                        # try:
                            
                            # البحث عن عنصر input الذي يقبل الصور
                        #     file_input = self.page.locator('div[role="textbox"][contenteditable="true"]').first
                        #     # file_input.wait_for(state="visible", timeout=5000)

                        #     # رفع الصورة مباشرة
                        #     file_input.set_input_files(image_path)
                        #     logging.info("✅ تم رفع الصورة بنجاح.")
                        #     time.sleep(3)  # مهلة صغيرة بعد رفع الصورة

                        # except Exception as e:
                        #     logging.error(f"❌ فشل رفع الصورة تلقائيًا: {e}")



                # # 6. رفع الصورة إن وجدت
                # if self.add_image_var.get() and hasattr(self, 'selected_image'):
                #     for sel in ["//div[@aria-label='صورة/فيديو']", "//div[@aria-label='Photo/video']", "//input[@accept='image/*']"]:
                #         try:
                #             elem = self.page.locator(sel).first

                #             # استخدام wait_for بدلاً من is_visible
                #             elem.wait_for(state="visible", timeout=3000)

                #             if sel.startswith('//input'):
                #                 elem.set_input_files(self.selected_image)
                #             else:
                #                 elem.click()
                #                 # انتظار ظهور input لرفع الصورة
                #                 self.page.locator("input[type='file']").first.wait_for(state="visible", timeout=3000)
                #                 self.page.locator("input[type='file']").first.set_input_files(self.selected_image)

                #             time.sleep(5)
                #             break  # توقف بعد أول نجاح
                #         except Exception as e:
                #             logging.debug(f"فشل رفع الصورة باستخدام المحدد {sel}: {e}")
                #             continue



                # 7. النقر على زر النشر
                # clicked = False
                # for sel in ["//div[@aria-label='نشر']", "//div[@aria-label='Post']", "//span[text()='نشر']", "//span[text()='Post']"]:
                #     try:
                #         btn = self.page.locator(sel).first
                #         if btn.is_visible(timeout=3000):
                #             # btn.click()
                #             clicked = True
                #             break
                #     except:
                #         continue

                if not clicked:
                    raise Exception("لم يتم العثور على زر النشر")

                # time.sleep(3)
                # تحديث الحالة بعد النشر
                current_time = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
                self.update_group_status(item, "تم النشر", current_time, str(current_retry + 1))
                return True

            except Exception as e:
                current_retry += 1
                logging.error(f"Attempt {current_retry}/{max_retries} failed for {group_data[0]}: {e}")
                if current_retry < max_retries and self.is_publishing:
                    self.update_group_status(item, f"محاولة {current_retry + 1}", "-", str(current_retry))
                    time.sleep(5)
                else:
                    self.update_group_status(item, "فشل النشر", "-", str(current_retry))
                    # if self.is_publishing:
                        # print("no-po..")
                        # messagebox.showwarning(
                        #     "تنبيه",
                        #     f"فشل النشر في المجموعة {group_data[0]} بعد {max_retries} محاولات"
                        # )
        return False


# --------------------------------------------------------------------------------------------------



# --------------------------------------------------------------------------------------------------




    def on_closing(self):
        """معالجة إغلاق النافذة"""
        if self.is_publishing:
            if messagebox.askokcancel("تأكيد", "هل تريد إيقاف النشر وإغلاق البرنامج؟"):
                self.stop_publishing()
                self.window.destroy()
        else:
            self.window.destroy()

    def run(self):
        """تشغيل التطبيق"""
        self.window.mainloop()

if __name__ == "__main__":
    app = FacebookAutomationApp()
    app.run()
