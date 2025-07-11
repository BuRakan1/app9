class ModernTrainingDesign(tk.Tk):
    """تطبيق إدارة التدريب - قالب فارغ"""

    APP_NAME = "نظام إدارة التدريب"
    BASE_PATH = getattr(sys, "_MEIPASS", os.path.dirname(os.path.abspath(__file__)))
    FONT_PATH = os.path.join(BASE_PATH, "assets", "fonts", "Tajawal-Regular.ttf")

    # نظام الألوان
    COLORS = {
        "primary": "#1a73e8",
        "secondary": "#4285f4",
        "success": "#34a853",
        "danger": "#ea4335",
        "warning": "#fbbc05",
        "light": "#f0f4f8",
        "dark": "#202124",
        "surface": "#FFFFFF",
        "background": "#f0f4f8",
        "card": "#FFFFFF",
        "on_surface": "#202124",
        "border": "#E5E7EB",
    }

    # — حدّث/أضف داخل القاموس FONTS —
    FONTS = {
        "large_title": ("Tajawal", 24, "bold"),
        "title": ("Tajawal", 18, "bold"),
        "subtitle": ("Tajawal", 16, "bold"),
        "text": ("Tajawal", 12),
        "text_bold": ("Tajawal", 12, "bold"),
        "small": ("Tajawal", 10),
        "nav": ("Tajawal", 20, "bold"),  # ← خذ مقاس أكبر للزرار العلوية
        "body": ("Tajawal", 14),
        "large": ("Tajawal", 16, "bold"),
    }

    def __init__(self, root=None, current_user=None, db_conn=None):
        super().__init__()

        self.title("نظام إدارة التدريب - قسم تصميم وتطوير البرامج")
        self.current_user = current_user
        self.db_conn = db_conn

        # التكيف مع حجم الشاشة تلقائياً
        screen_width = self.winfo_screenwidth()
        screen_height = self.winfo_screenheight()

        app_width = min(1400, int(screen_width * 0.90))
        app_height = min(800, int(screen_height * 0.90))

        x = (screen_width - app_width) // 2
        y = (screen_height - app_height) // 2

        self.geometry(f"{app_width}x{app_height}+{x}+{y}")
        self.minsize(800, 600)
        self.configure(bg=self.COLORS["background"])

        self.screen_width = screen_width
        self.screen_height = screen_height
        self.app_width = app_width
        self.app_height = app_height

        self._load_font()
        self._setup_styles()

        self.current_page = "الرئيسية"
        self._is_closing = False  # متغير لتتبع حالة الإغلاق

        # إنشاء الجداول إذا لم تكن موجودة
        self._create_database_tables()

        # إنشاء عناصر الواجهة
        self._create_header()
        self._create_tab_control()
        self._create_footer()

        # معالجة إغلاق النافذة بشكل صحيح
        self.protocol("WM_DELETE_WINDOW", self._on_closing)

    def _on_closing(self):
        """معالجة إغلاق النافذة بشكل نظيف"""
        if self._is_closing:
            return

        self._is_closing = True

        try:
            # إلغاء جميع الأحداث المجدولة
            for widget in self.winfo_children():
                try:
                    widget.destroy()
                except:
                    pass

            # إغلاق اتصال قاعدة البيانات
            if hasattr(self, 'db_conn') and self.db_conn:
                try:
                    self.db_conn.close()
                except:
                    pass

            # إيقاف mainloop
            self.quit()

        except:
            pass
        finally:
            try:
                self.destroy()
            except:
                pass

            # إنهاء البرنامج
            import sys
            sys.exit(0)

    def _load_font(self):
        """تحميل خط Tajawal"""
        if os.path.isfile(self.FONT_PATH):
            try:
                tkfont.nametofont("Tajawal")
            except tk.TclError:
                self.tk.call("font", "create", "Tajawal", "-family", "Tajawal", "-size", 12)
                self.tk.call("font", "configure", "Tajawal", "-file", self.FONT_PATH)

    def _setup_styles(self):
        """تبويبات رمادية داكنة، تكبير بسيط، خط أسود"""
        try:
            style = ttk.Style(self)

            # التحقق من توفر الثيم
            available_themes = style.theme_names()
            if "clam" in available_themes:
                style.theme_use("clam")
            else:
                # استخدام الثيم الافتراضي إذا لم يكن clam متوفراً
                style.theme_use("default")

            # ألوان مطلوبة
            base_gray = "#8c8c8c"  # رمادي أغمق للزر غير المحدَّد
            hover_gray = "#a8a8a8"  # رمادي متوسط عند المرور
            selected_color = "#3B82F6"  # أزرق للتبويبة المحددة
            txt_color = "black"
            selected_txt = "black"  # نص أسود للتبويبة المحددة
            notebook_bg = "#e8e8e8"  # رمادي فاتح للخلفية

            # Notebook بخلفية رمادية فاتحة
            style.configure("Bold.TNotebook",
                            background=notebook_bg,  # تغيير الخلفية إلى رمادي فاتح
                            borderwidth=0,
                            relief="flat",
                            tabmargins=[2, 5, 2, 0])  # إضافة مسافات بين التبويبات

            # تبويبات أكبر قليلاً: خط أكبر وبادينغ أكثر
            style.configure("Bold.TNotebook.Tab",
                            font=self.FONTS["subtitle"],  # ← حجم 16 عريض
                            padding=[16, 8],  # ← عرض وارتفاع أكبر
                            borderwidth=0,
                            relief="flat",
                            background=base_gray,
                            foreground=txt_color)

            # تغيّر اللون عند الاختيار أو المرور
            style.map("Bold.TNotebook.Tab",
                      background=[("selected", selected_color),
                                  ("active", hover_gray)],
                      foreground=[("selected", selected_txt),
                                  ("active", txt_color)])

            # إزالة خط التركيز نهائيًا
            try:
                style.layout("Bold.TNotebook.Tab",
                             [('Notebook.padding', {'side': 'top', 'sticky': 'nswe', 'children': [
                                 ('Notebook.label', {'side': 'top', 'sticky': ''})
                             ]})])
            except:
                # إذا فشل تخصيص التخطيط، نتجاهل الخطأ
                pass

        except Exception as e:
            # في حالة حدوث أي خطأ، نستمر بدون تخصيص الأنماط
            print(f"خطأ في إعداد الأنماط: {e}")

    def _create_header(self):
        """إلغاء الشريط الأزرق العلوي نهائيًّا."""
        return  # ما نرسم أي إطار أو أزرار

    def _cleanup_window(self, window):
        """تنظيف الذاكرة عند إغلاق النافذة"""
        try:
            # إلغاء جميع الأحداث المجدولة
            for after_id in window.tk.call('after', 'info'):
                window.after_cancel(after_id)

            # تدمير جميع العناصر الفرعية
            for widget in window.winfo_children():
                widget.destroy()

            # تدمير النافذة
            window.destroy()

            # استدعاء جامع القمامة
            import gc
            gc.collect()

        except:
            pass

    def _create_tab_control(self):
        """إنشاء التبويبات"""
        # إطار التبويبات
        tab_frame = tk.Frame(self, bg=self.COLORS["background"])
        tab_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        self.tab_control = ttk.Notebook(tab_frame, style="Bold.TNotebook")
        self.tab_control.pack(fill=tk.BOTH, expand=True)

        # إنشاء التبويبات
        self._create_home_tab()
        self._create_teachers_tab()
        self._create_courses_tab()
        if self.current_user and self.current_user["permissions"]["is_admin"]:
            self._create_users_tab()
        self._create_settings_tab()

    def _create_footer(self):
        """شريط سفلي محسّن مع معلومات النظام وزر الخروج"""
        footer = tk.Frame(self, bg="#2c3e50", height=50)
        footer.pack(side=tk.BOTTOM, fill=tk.X)
        footer.pack_propagate(False)

        # الجانب الأيسر - معلومات المستخدم والنظام
        left_frame = tk.Frame(footer, bg="#2c3e50")
        left_frame.pack(side=tk.LEFT, padx=20, pady=8)

        # أيقونة المستخدم
        user_icon = tk.Label(
            left_frame,
            text="👤",
            font=("Arial", 16),
            bg="#2c3e50",
            fg="white"
        )
        user_icon.pack(side=tk.LEFT, padx=(0, 10))

        # معلومات المستخدم
        user_name = tk.Label(
            left_frame,
            text=f"المستخدم: {self.current_user['full_name']}" if self.current_user else "غير معرف",
            font=self.FONTS["text_bold"],
            bg="#2c3e50",
            fg="white"
        )
        user_name.pack(side=tk.LEFT)

        # الوسط - حالة النظام
        center_frame = tk.Frame(footer, bg="#2c3e50")
        center_frame.pack(side=tk.LEFT, expand=True, padx=20)

        status_text = tk.Label(
            center_frame,
            text="إدارة تصميم و تطوير البرامج",
            font=self.FONTS["title"],  # خط أكبر
            bg="#2c3e50",
            fg="white"  # لون أبيض
        )
        status_text.pack(side=tk.LEFT)

        # الجانب الأيمن - زر الخروج
        right_frame = tk.Frame(footer, bg="#2c3e50")
        right_frame.pack(side=tk.RIGHT, padx=20, pady=8)

        # زر تسجيل الخروج
        logout_btn = tk.Button(
            right_frame,
            text="⚡ تسجيل الخروج",
            font=self.FONTS["text_bold"],
            bg="#e74c3c",
            fg="white",
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            padx=20,
            pady=8,
            command=self._logout
        )
        logout_btn.pack(side=tk.RIGHT)

        # تأثير عند المرور على زر الخروج
        def on_enter(e):
            logout_btn['bg'] = '#c0392b'

        def on_leave(e):
            logout_btn['bg'] = '#e74c3c'

        logout_btn.bind("<Enter>", on_enter)
        logout_btn.bind("<Leave>", on_leave)

    def _create_home_tab(self):
        """إنشاء تبويب الصفحة الرئيسية - فارغ"""
        home_frame = tk.Frame(self.tab_control, bg=self.COLORS["background"])
        self.tab_control.add(home_frame, text="الرئيسية")

        # محتوى فارغ
        tk.Label(
            home_frame,
            text="الصفحة الرئيسية",
            font=self.FONTS["large_title"],
            bg=self.COLORS["background"],
            fg=self.COLORS["dark"]
        ).pack(pady=50)

    #— ضفها داخل ModernTrainingDesign —
    def _logout(self):
        """تأكيد الخروج ثم إغلاق النافذة الرئيسية."""
        if messagebox.askyesno("تأكيد", "هل تريد تسجيل الخروج؟"):
            self._on_closing()

    def _create_teachers_tab(self):
        """إنشاء تبويب إدارة المدرسين بتصميم محسن"""
        teachers_frame = tk.Frame(self.tab_control, bg=self.COLORS["background"])
        self.tab_control.add(teachers_frame, text="إدارة المدرسين")

        # إطار العنوان والأزرار
        header_frame = tk.Frame(teachers_frame, bg="#1E3A5F", height=80)  # أزرق داكن رسمي
        header_frame.pack(fill=tk.X, padx=10, pady=10)
        header_frame.pack_propagate(False)

        # عنوان الصفحة
        title_label = tk.Label(
            header_frame,
            text="إدارة المدرسين",
            font=self.FONTS["large_title"],
            bg="#1E3A5F",  # نفس لون الخلفية
            fg="white"  # نص أبيض
        )
        title_label.pack(side=tk.LEFT, padx=20, pady=20)

        # إطار الأزرار
        buttons_frame = tk.Frame(header_frame, bg="#1E3A5F")  # نفس اللون الرسمي
        buttons_frame.pack(side=tk.RIGHT, padx=20, pady=20)

        # زر إضافة مدرس
        add_teacher_btn = tk.Button(
            buttons_frame,
            text="إضافة مدرس جديد",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["success"],
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._open_add_teacher_window
        )
        add_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر تعديل مدرس
        edit_teacher_btn = tk.Button(
            buttons_frame,
            text="تعديل بيانات مدرس",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["warning"],
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._edit_teacher
        )
        edit_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر حذف مدرس
        delete_teacher_btn = tk.Button(
            buttons_frame,
            text="حذف مدرس",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["danger"],
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._delete_teacher
        )
        delete_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر استيراد مدرسين
        import_teacher_btn = tk.Button(
            buttons_frame,
            text="استيراد مدرسين",
            font=self.FONTS["text_bold"],
            bg="#9C27B0",  # لون بنفسجي
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._import_teachers
        )
        import_teacher_btn.pack(side=tk.LEFT, padx=5)

        # زر مسارات مدرسي الدورات - محدث
        paths_btn = tk.Button(
            buttons_frame,
            text="📚 مسارات مدرسي الدورات",
            font=self.FONTS["text_bold"],
            bg="#17a2b8",  # لون أزرق فاتح
            fg="white",
            padx=20,
            pady=10,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._manage_course_teachers_paths  # الدالة الجديدة
        )
        paths_btn.pack(side=tk.LEFT, padx=5)

        # إطار البحث والتصفية
        search_filter_frame = tk.Frame(teachers_frame, bg=self.COLORS["surface"], height=70)
        search_filter_frame.pack(fill=tk.X, padx=15, pady=(10, 5))
        search_filter_frame.pack_propagate(False)

        # إطار داخلي للمحتوى مع توزيع ثابت
        inner_frame = tk.Frame(search_filter_frame, bg=self.COLORS["surface"])
        inner_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

        # تكوين الأعمدة بأوزان ثابتة
        inner_frame.grid_columnconfigure(0, weight=1)  # البحث
        inner_frame.grid_columnconfigure(1, weight=1)  # الفلتر
        inner_frame.grid_columnconfigure(2, weight=0)  # زر التصدير

        # 1. إطار البحث (ثابت في اليسار)
        search_container = tk.Frame(inner_frame, bg=self.COLORS["surface"])
        search_container.grid(row=0, column=0, sticky="w", padx=(0, 20))

        tk.Label(
            search_container,
            text="بحث:",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        self.search_entry = tk.Entry(
            search_container,
            font=self.FONTS["text"],
            width=25
        )
        self.search_entry.pack(side=tk.LEFT)

        # ربط حدث الكتابة مباشرة
        def on_search_key(event):
            # تأخير بسيط لتحسين الأداء
            self.after(100, self._apply_filters)

        self.search_entry.bind('<KeyRelease>', on_search_key)

        # 2. إطار التصفية (في الوسط)
        filter_container = tk.Frame(inner_frame, bg=self.COLORS["surface"])
        filter_container.grid(row=0, column=1, sticky="w")

        tk.Label(
            filter_container,
            text="الفئة:",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        # في _create_teachers_tab، تأكد من وجود:
        self.filter_combo = ttk.Combobox(
            filter_container,
            values=["الكل", "منسوبي المدينة", "متعاونين"],
            font=self.FONTS["text"],
            width=20,
            state="readonly"
        )
        self.filter_combo.pack(side=tk.LEFT)
        self.filter_combo.set("الكل")
        self.filter_combo.bind('<<ComboboxSelected>>', lambda e: self._apply_filters())

        # 3. زر التصدير (ثابت في اليمين)
        self.export_btn = tk.Button(
            inner_frame,
            text="📊 تصدير البيانات",
            font=self.FONTS["text_bold"],
            bg="#4CAF50",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=self._export_filtered_data
        )
        self.export_btn.grid(row=0, column=2, sticky="e")

        # خط فاصل
        separator = tk.Frame(teachers_frame, bg=self.COLORS["border"], height=2)
        separator.pack(fill=tk.X, padx=15, pady=(0, 10))

        # إطار الجدول الرئيسي
        main_table_frame = tk.Frame(teachers_frame, bg="#FFFFFF", bd=2, relief=tk.RIDGE)
        main_table_frame.pack(fill=tk.BOTH, expand=True, padx=15, pady=(0, 10))

        # إطار داخلي للجدول
        table_frame = tk.Frame(main_table_frame, bg="#FFFFFF")
        table_frame.pack(fill=tk.BOTH, expand=True, padx=3, pady=3)

        # إنشاء Treeview بتصميم رسمي
        style = ttk.Style()

        # تكوين نمط الجدول
        style.configure("Formal.Treeview",
                        background="#FFFFFF",
                        foreground="#000000",
                        rowheight=45,
                        fieldbackground="#FFFFFF",
                        font=("Tajawal", 14, "normal"),
                        borderwidth=1,
                        relief="solid")

        # تكوين رؤوس الأعمدة
        style.configure("Formal.Treeview.Heading",
                        font=("Tajawal", 16, "bold"),
                        background="#1E3A5F",  # أزرق داكن رسمي
                        foreground="#FFFFFF",
                        relief="raised",
                        borderwidth=1,
                        padding=[10, 8])

        # تكوين التحديد
        style.map('Formal.Treeview',
                  background=[('selected', '#4682B4'),  # أزرق رسمي
                              ('active', '#E6F2FF')],  # أزرق فاتح جداً
                  foreground=[('selected', '#FFFFFF'),
                              ('active', '#000000')])

        # شريط التمرير
        style.configure("Formal.Vertical.TScrollbar",
                        background="#D3D3D3",
                        troughcolor="#F5F5F5",
                        borderwidth=1,
                        arrowcolor="#696969",
                        width=16)

        # شريط التمرير العمودي
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical", style="Formal.Vertical.TScrollbar")
        v_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # شريط التمرير الأفقي
        h_scrollbar = ttk.Scrollbar(table_frame, orient="horizontal")
        h_scrollbar.pack(side=tk.BOTTOM, fill=tk.X)

        # إنشاء الجدول - تم حذف عمود id
        self.teachers_tree = ttk.Treeview(
            table_frame,
            columns=("name", "rank", "id_number", "workplace", "qualification", "phone"),
            show="tree headings",
            style="Formal.Treeview",
            yscrollcommand=v_scrollbar.set,
            xscrollcommand=h_scrollbar.set,
            height=12
        )

        # إخفاء عمود الشجرة
        self.teachers_tree.column("#0", width=0, stretch=tk.NO)

        # تكوين الأعمدة - بدون عمود الرقم
        column_configs = [
            ("name", "الاسم الكامل", 350, tk.CENTER),
            ("rank", "الرتبة العسكرية", 220, tk.CENTER),
            ("id_number", "رقم الهوية", 200, tk.CENTER),
            ("workplace", "جهة العمل", 350, tk.CENTER),
            ("qualification", "المؤهل الدراسي", 280, tk.CENTER),
            ("phone", "رقم الجوال", 180, tk.CENTER)
        ]

        for col_id, heading, width, anchor in column_configs:
            self.teachers_tree.column(col_id, width=width, anchor=anchor, minwidth=width - 30)
            self.teachers_tree.heading(col_id, text=heading, anchor=tk.CENTER)

        # تكوين ألوان الصفوف
        self.teachers_tree.tag_configure('oddrow',
                                         background='#FFFFFF',
                                         font=("Tajawal", 13, "normal"))
        self.teachers_tree.tag_configure('evenrow',
                                         background='#F0F8FF',  # أزرق فاتح جداً
                                         font=("Tajawal", 13, "normal"))
        self.teachers_tree.tag_configure('highlight',
                                         background='#FFE4B5',  # بيج فاتح
                                         font=("Tajawal", 13, "bold"))

        self.teachers_tree.pack(fill=tk.BOTH, expand=True)
        v_scrollbar.config(command=self.teachers_tree.yview)
        h_scrollbar.config(command=self.teachers_tree.xview)

        # إطار المعلومات السفلي
        info_frame = tk.Frame(teachers_frame, bg="#1E3A5F", height=60)
        info_frame.pack(fill=tk.X, padx=15, pady=(5, 10))
        info_frame.pack_propagate(False)

        # خط فاصل علوي
        separator = tk.Frame(info_frame, bg="#FFFFFF", height=2)
        separator.pack(fill=tk.X)

        # إطار داخلي للمعلومات
        inner_info = tk.Frame(info_frame, bg="#1E3A5F")
        inner_info.pack(expand=True)

        self.teacher_count_label = tk.Label(
            inner_info,
            text="إجمالي المدرسين: 0 | منسوبي المدينة: 0 | المتعاونين: 0",
            font=("Tajawal", 14, "bold"),
            bg="#1E3A5F",
            fg="#FFFFFF"
        )
        self.teacher_count_label.pack(pady=15)

        # تحميل بيانات المدرسين
        self._load_teachers()

        # ربط الأحداث
        self.teachers_tree.bind("<Double-Button-1>", self._on_teacher_double_click)
        self.teachers_tree.bind("<Button-3>", self._show_context_menu)

    # إصلاحات لنظام إدارة هيئة التدريس

    # إصلاحات لنظام إدارة هيئة التدريس

    def _manage_course_teachers_paths(self):
        """إدارة مسارات هيئة التدريس"""
        paths_window = tk.Toplevel(self)
        paths_window.title("مسارات هيئة التدريس")
        paths_window.geometry("1200x700")
        paths_window.configure(bg=self.COLORS["background"])

        # السماح بتكبير النافذة
        paths_window.resizable(True, True)
        paths_window.minsize(1000, 600)
        paths_window.state('normal')

        # توسيط النافذة
        paths_window.update_idletasks()
        x = (paths_window.winfo_screenwidth() - 1200) // 2
        y = (paths_window.winfo_screenheight() - 700) // 2
        paths_window.geometry(f"1200x700+{x}+{y}")

        # شريط العنوان
        header_frame = tk.Frame(paths_window, bg="#1E3A5F", height=80)
        header_frame.pack(fill=tk.X)
        header_frame.pack_propagate(False)

        header_content = tk.Frame(header_frame, bg="#1E3A5F")
        header_content.pack(expand=True, fill=tk.BOTH, padx=30)

        title_label = tk.Label(
            header_content,
            text="مسارات هيئة التدريس",
            font=("Tajawal", 24, "bold"),
            bg="#1E3A5F",
            fg="white"
        )
        title_label.pack(side=tk.LEFT, pady=20)

        # إطار الأزرار في الشريط العلوي
        header_buttons = tk.Frame(header_content, bg="#1E3A5F")
        header_buttons.pack(side=tk.RIGHT, pady=20)

        # زر إضافة دورة
        add_course_btn = tk.Button(
            header_buttons,
            text="إضافة دورة",
            font=("Tajawal", 13, "bold"),
            bg="#4CAF50",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2"
        )
        add_course_btn.pack(side=tk.LEFT, padx=(0, 10))

        # زر حذف دورة
        delete_course_btn = tk.Button(
            header_buttons,
            text="حذف دورة",
            font=("Tajawal", 13, "bold"),
            bg="#dc3545",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2"
        )
        delete_course_btn.pack(side=tk.LEFT, padx=(0, 10))

        # زر تصدير البيانات
        export_btn = tk.Button(
            header_buttons,
            text="تصدير بيانات هيئة التدريس",
            font=("Tajawal", 13, "bold"),
            bg="#FF9800",
            fg="white",
            padx=20,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2"
        )
        export_btn.pack(side=tk.LEFT)

        # إطار البحث
        search_frame = tk.Frame(paths_window, bg=self.COLORS["surface"], height=70)
        search_frame.pack(fill=tk.X, padx=15, pady=(0, 10))
        search_frame.pack_propagate(False)

        search_container = tk.Frame(search_frame, bg=self.COLORS["surface"])
        search_container.pack(side=tk.LEFT, padx=20, pady=20)

        tk.Label(
            search_container,
            text="البحث عن دورة:",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["surface"]
        ).pack(side=tk.LEFT, padx=(0, 10))

        search_entry = tk.Entry(
            search_container,
            font=("Tajawal", 13),
            width=30,
            bd=2,
            relief=tk.FLAT,
            highlightthickness=2,
            highlightcolor="#1E3A5F"
        )
        search_entry.pack(side=tk.LEFT)

        # خط فاصل
        separator = tk.Frame(paths_window, bg=self.COLORS["border"], height=2)
        separator.pack(fill=tk.X, padx=15, pady=(0, 10))

        # إطار الجدول الرئيسي
        main_table_frame = tk.Frame(paths_window, bg="#FFFFFF", bd=2, relief=tk.RIDGE)
        main_table_frame.pack(fill=tk.BOTH, expand=True, padx=15, pady=(0, 10))

        table_frame = tk.Frame(main_table_frame, bg="#FFFFFF")
        table_frame.pack(fill=tk.BOTH, expand=True, padx=3, pady=3)

        # شريط التمرير
        v_scrollbar = ttk.Scrollbar(table_frame, orient="vertical")
        v_scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

        # تنسيق Treeview
        style = ttk.Style()
        style.configure("Paths.Treeview",
                        background="#FFFFFF",
                        foreground="#000000",
                        rowheight=45,
                        fieldbackground="#FFFFFF",
                        font=("Tajawal", 14, "normal"),
                        borderwidth=1,
                        relief="solid")

        style.configure("Paths.Treeview.Heading",
                        font=("Tajawal", 16, "bold"),
                        background="#1E3A5F",
                        foreground="#FFFFFF",
                        relief="raised",
                        borderwidth=1,
                        padding=[10, 8])

        style.map('Paths.Treeview',
                  background=[('selected', '#4682B4'),
                              ('active', '#E6F2FF')],
                  foreground=[('selected', '#FFFFFF'),
                              ('active', '#000000')])

        # إنشاء الجدول
        paths_tree = ttk.Treeview(
            table_frame,
            columns=("course_name", "responsible_teacher", "substitute_teachers", "total"),
            show="tree headings",
            style="Paths.Treeview",
            yscrollcommand=v_scrollbar.set,
            height=12
        )

        # إخفاء عمود الشجرة
        paths_tree.column("#0", width=0, stretch=tk.NO)

        # تكوين الأعمدة
        column_configs = [
            ("course_name", "اسم الدورة", 400, tk.CENTER),
            ("responsible_teacher", "رئيس البرنامج التدريبي", 250, tk.CENTER),
            ("substitute_teachers", "هيئة التدريس", 350, tk.CENTER),
            ("total", "العدد الكلي", 120, tk.CENTER)
        ]

        for col_id, heading, width, anchor in column_configs:
            paths_tree.column(col_id, width=width, anchor=anchor, minwidth=width - 30)
            paths_tree.heading(col_id, text=heading, anchor=tk.CENTER)

        # تكوين ألوان الصفوف
        paths_tree.tag_configure('oddrow',
                                 background='#FFFFFF',
                                 font=("Tajawal", 13, "normal"))
        paths_tree.tag_configure('evenrow',
                                 background='#F0F8FF',
                                 font=("Tajawal", 13, "normal"))

        paths_tree.pack(fill=tk.BOTH, expand=True)
        v_scrollbar.config(command=paths_tree.yview)

        # دالة تحميل البيانات
        def load_course_paths(search_term=""):
            # مسح الجدول
            for item in paths_tree.get_children():
                paths_tree.delete(item)

            try:
                cursor = self.db_conn.cursor()

                # الحصول على جميع الدورات من course_names
                if search_term:
                    cursor.execute("""
                        SELECT DISTINCT name 
                        FROM course_names 
                        WHERE is_active = 1 AND LOWER(name) LIKE LOWER(?)
                        ORDER BY name
                    """, (f'%{search_term}%',))
                else:
                    cursor.execute("""
                        SELECT DISTINCT name 
                        FROM course_names 
                        WHERE is_active = 1
                        ORDER BY name
                    """)

                all_courses = cursor.fetchall()

                for index, (course_name,) in enumerate(all_courses):
                    # الحصول على المدرس المسؤول
                    cursor.execute("""
                        SELECT t.name 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 1
                    """, (course_name,))

                    responsible = cursor.fetchone()
                    responsible_name = responsible[0] if responsible else "غير محدد"

                    # الحصول على هيئة التدريس - منسوبي المدينة
                    cursor.execute("""
                        SELECT t.name 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category = 'منسوبي المدينة'
                        ORDER BY t.name
                    """, (course_name,))

                    city_staff = cursor.fetchall()

                    # الحصول على هيئة التدريس - المتعاونين
                    cursor.execute("""
                        SELECT t.name 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY t.name
                    """, (course_name,))

                    collaborators = cursor.fetchall()

                    # تجميع الأعضاء
                    all_members = []
                    if city_staff:
                        all_members.append(
                            f"منسوبي المدينة ({len(city_staff)}): " + ", ".join([s[0] for s in city_staff]))
                    if collaborators:
                        all_members.append(
                            f"المتعاونين ({len(collaborators)}): " + ", ".join([c[0] for c in collaborators]))

                    substitute_names = " | ".join(all_members) if all_members else "لا يوجد"

                    # العدد الكلي
                    total_count = len(city_staff) + len(collaborators) + (1 if responsible else 0)

                    # تحديد التاج
                    tag = 'evenrow' if index % 2 == 0 else 'oddrow'

                    # إضافة للجدول
                    paths_tree.insert("", tk.END, values=(
                        course_name,
                        responsible_name,
                        substitute_names,
                        total_count
                    ), tags=(tag,))

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ في تحميل البيانات: {str(e)}")

        # دالة إضافة دورة جديدة محدثة
        def add_course_to_paths():
            """إضافة دورة جديدة لمسارات هيئة التدريس"""
            add_dialog = tk.Toplevel(paths_window)
            add_dialog.title("إضافة دورة")
            add_dialog.geometry("600x500")
            add_dialog.configure(bg=self.COLORS["surface"])
            add_dialog.transient(paths_window)
            add_dialog.grab_set()

            # توسيط النافذة
            add_dialog.update_idletasks()
            x = (add_dialog.winfo_screenwidth() - 600) // 2
            y = (add_dialog.winfo_screenheight() - 500) // 2
            add_dialog.geometry(f"600x500+{x}+{y}")

            # العنوان
            tk.Label(
                add_dialog,
                text="إضافة دورة لهيئة التدريس",
                font=("Tajawal", 16, "bold"),
                bg=self.COLORS["primary"],
                fg="white",
                pady=20
            ).pack(fill=tk.X)

            # إطار المحتوى
            content = tk.Frame(add_dialog, bg=self.COLORS["surface"], padx=30, pady=20)
            content.pack(fill=tk.BOTH, expand=True)

            # البحث
            tk.Label(
                content,
                text="البحث عن دورة:",
                font=("Tajawal", 12, "bold"),
                bg=self.COLORS["surface"]
            ).pack(anchor=tk.W, pady=(0, 10))

            search_frame = tk.Frame(content, bg=self.COLORS["surface"])
            search_frame.pack(fill=tk.X, pady=(0, 20))

            search_entry = tk.Entry(
                search_frame,
                font=("Tajawal", 12),
                bd=2,
                relief=tk.FLAT,
                highlightthickness=2,
                highlightcolor=self.COLORS["primary"]
            )
            search_entry.pack(fill=tk.X)

            # قائمة الدورات
            list_frame = tk.Frame(content, bg="white", relief=tk.GROOVE, bd=2)
            list_frame.pack(fill=tk.BOTH, expand=True)

            scrollbar = tk.Scrollbar(list_frame)
            scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

            courses_listbox = tk.Listbox(
                list_frame,
                font=("Tajawal", 12),
                yscrollcommand=scrollbar.set,
                selectmode=tk.SINGLE
            )
            courses_listbox.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=courses_listbox.yview)

            # متغير لتخزين جميع الدورات المتاحة
            all_available_courses = []
            course_names_map = {}  # لربط الأسماء بالمعرفات

            # تحميل الدورات المتاحة
            def load_available_courses():
                cursor = self.db_conn.cursor()

                # الحصول على جميع الدورات من course_names
                cursor.execute("""
                    SELECT DISTINCT name 
                    FROM course_names 
                    WHERE is_active = 1 
                    ORDER BY name
                """)

                all_courses = cursor.fetchall()

                # الحصول على الدورات التي لها مدرسين بالفعل
                cursor.execute("""
                    SELECT DISTINCT course_name 
                    FROM course_teacher_paths
                """)
                existing_courses = set([c[0] for c in cursor.fetchall()])

                # تحديد الدورات المتاحة (غير الموجودة في مسارات هيئة التدريس)
                all_available_courses.clear()
                course_names_map.clear()

                for i, (course_name,) in enumerate(all_courses):
                    if course_name not in existing_courses:
                        all_available_courses.append(course_name)
                        course_names_map[i] = course_name

                # عرض جميع الدورات المتاحة
                update_course_list("")

            def update_course_list(search_text):
                """تحديث قائمة الدورات بناءً على البحث"""
                courses_listbox.delete(0, tk.END)

                search_text = search_text.strip().lower()

                for course_name in all_available_courses:
                    if not search_text or search_text in course_name.lower():
                        courses_listbox.insert(tk.END, course_name)

            # ربط البحث
            def on_search_change(event=None):
                update_course_list(search_entry.get())

            search_entry.bind('<KeyRelease>', on_search_change)

            # إطار الأزرار
            btn_frame = tk.Frame(add_dialog, bg=self.COLORS["surface"])
            btn_frame.pack(fill=tk.X, pady=20)

            def confirm_add():
                selection = courses_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار دورة")
                    return

                # الحصول على اسم الدورة المحددة فقط
                selected_course_name = courses_listbox.get(selection[0])

                try:
                    cursor = self.db_conn.cursor()

                    # التحقق مرة أخرى من أن الدورة غير موجودة
                    cursor.execute("""
                        SELECT COUNT(*) FROM course_teacher_paths 
                        WHERE course_name = ?
                    """, (selected_course_name,))

                    if cursor.fetchone()[0] > 0:
                        messagebox.showwarning("تنبيه", "هذه الدورة موجودة بالفعل في هيئة التدريس")
                        return

                    # إغلاق النافذة
                    add_dialog.destroy()

                    # عرض رسالة النجاح
                    messagebox.showinfo("نجاح",
                                        f"تمت إضافة دورة '{selected_course_name}' لهيئة التدريس\n"
                                        "يمكنك الآن تعيين رئيس البرنامج التدريبي والأعضاء")

                    # تحديث القائمة الرئيسية
                    load_course_paths()

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            tk.Button(
                btn_frame,
                text="إضافة",
                font=("Tajawal", 12, "bold"),
                bg=self.COLORS["success"],
                fg="white",
                bd=0,
                padx=30,
                pady=8,
                cursor="hand2",
                command=confirm_add
            ).pack(side=tk.LEFT, padx=20)

            tk.Button(
                btn_frame,
                text="إلغاء",
                font=("Tajawal", 12, "bold"),
                bg=self.COLORS["danger"],
                fg="white",
                bd=0,
                padx=30,
                pady=8,
                cursor="hand2",
                command=add_dialog.destroy
            ).pack(side=tk.RIGHT, padx=20)

            # تحميل الدورات الأولية
            load_available_courses()

        # دالة حذف دورة من هيئة التدريس
        def delete_course_from_paths():
            """حذف دورة من مسارات هيئة التدريس مع جميع أعضائها"""
            selection = paths_tree.selection()
            if not selection:
                messagebox.showinfo("تنبيه", "يرجى اختيار دورة لحذفها")
                return

            item = paths_tree.item(selection[0])
            course_name = item['values'][0]

            # التأكيد
            result = messagebox.askyesno(
                "تأكيد الحذف",
                f"هل تريد حذف دورة '{course_name}' من هيئة التدريس؟\n\n"
                "سيتم حذف:\n"
                "• رئيس البرنامج التدريبي\n"
                "• جميع أعضاء هيئة التدريس\n\n"
                "ملاحظة: لن يتم حذف الدورة من مسميات الدورات"
            )

            if result:
                try:
                    cursor = self.db_conn.cursor()

                    # حذف جميع السجلات المرتبطة بهذه الدورة من course_teacher_paths
                    cursor.execute("""
                        DELETE FROM course_teacher_paths 
                        WHERE course_name = ?
                    """, (course_name,))

                    self.db_conn.commit()

                    # حذف العنصر من الشجرة مباشرة
                    paths_tree.delete(selection[0])

                    messagebox.showinfo("نجاح", f"تم حذف دورة '{course_name}' من هيئة التدريس بنجاح")

                except Exception as e:
                    self.db_conn.rollback()
                    messagebox.showerror("خطأ", f"حدث خطأ أثناء الحذف: {str(e)}")

        # دالة إدارة مدرسي الدورة المحدثة - قسمين منفصلين
        def manage_course_teachers(event=None):
            """إدارة مدرسي الدورة مع قسمين منفصلين لمنسوبي المدينة والمتعاونين"""
            selection = paths_tree.selection()
            if not selection:
                return

            item = paths_tree.item(selection[0])
            course_name = item['values'][0]

            # نافذة إدارة مدرسي الدورة
            manage_window = tk.Toplevel(paths_window)
            manage_window.title(f"إدارة هيئة التدريس - {course_name}")
            manage_window.geometry("1200x750")
            manage_window.configure(bg="#F5F5F5")
            manage_window.resizable(True, True)
            manage_window.minsize(1100, 700)

            # توسيط النافذة
            manage_window.update_idletasks()
            x = (manage_window.winfo_screenwidth() - 1200) // 2
            y = (manage_window.winfo_screenheight() - 750) // 2
            manage_window.geometry(f"1200x750+{x}+{y}")

            # شريط العنوان الرئيسي
            header_frame = tk.Frame(manage_window, bg="#1E3A5F", height=70)
            header_frame.pack(fill=tk.X)
            header_frame.pack_propagate(False)

            header_content = tk.Frame(header_frame, bg="#1E3A5F")
            header_content.pack(expand=True)

            tk.Label(
                header_content,
                text="إدارة هيئة التدريس",
                font=("Tajawal", 20, "bold"),
                bg="#1E3A5F",
                fg="white"
            ).pack()

            tk.Label(
                header_content,
                text=f"دورة: {course_name}",
                font=("Tajawal", 14),
                bg="#1E3A5F",
                fg="#E0E0E0"
            ).pack()

            # إطار المحتوى الرئيسي
            main_container = tk.Frame(manage_window, bg="#F5F5F5")
            main_container.pack(fill=tk.BOTH, expand=True, padx=20, pady=15)

            # القسم الأول: المدرس المسؤول
            responsible_frame = tk.Frame(main_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            responsible_frame.pack(fill=tk.X, pady=(0, 15))

            # رأس قسم المسؤول
            resp_header = tk.Frame(responsible_frame, bg="#2C3E50", height=40)
            resp_header.pack(fill=tk.X)
            resp_header.pack_propagate(False)

            tk.Label(
                resp_header,
                text=f"رئيس البرنامج التدريبي لـ ({course_name})",
                font=("Tajawal", 14, "bold"),
                bg="#2C3E50",
                fg="white"
            ).pack(expand=True)

            # محتوى قسم المسؤول
            resp_content = tk.Frame(responsible_frame, bg="#FFFFFF", padx=20, pady=15)
            resp_content.pack(fill=tk.BOTH)

            # تقسيم أفقي
            resp_content.grid_columnconfigure(0, weight=1)
            resp_content.grid_columnconfigure(1, weight=1)

            # الجانب الأيمن - المسؤول الحالي
            right_resp = tk.Frame(resp_content, bg="#FFFFFF")
            right_resp.grid(row=0, column=1, sticky="nsew", padx=(10, 0))

            tk.Label(
                right_resp,
                text="الرئيس الحالي",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # إطار عرض المسؤول
            current_resp_frame = tk.Frame(right_resp, bg="#F8F9FA", relief=tk.RIDGE, bd=1, height=70)
            current_resp_frame.pack(fill=tk.X)
            current_resp_frame.pack_propagate(False)

            current_responsible_label = tk.Label(
                current_resp_frame,
                text="",
                font=("Tajawal", 12),
                bg="#F8F9FA",
                fg="#333333",
                justify=tk.CENTER
            )
            current_responsible_label.pack(expand=True)

            # زر حذف المسؤول
            remove_resp_btn = tk.Button(
                right_resp,
                text="حذف الرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_resp_btn.pack(pady=(10, 0))

            # الجانب الأيسر - البحث وتعيين
            left_resp = tk.Frame(resp_content, bg="#FFFFFF")
            left_resp.grid(row=0, column=0, sticky="nsew", padx=(0, 10))

            tk.Label(
                left_resp,
                text="تعيين رئيس جديد",
                font=("Tajawal", 13, "bold"),
                bg="#FFFFFF",
                fg="#2C3E50"
            ).pack(anchor=tk.E, pady=(0, 10))

            # البحث
            search_container = tk.Frame(left_resp, bg="#FFFFFF")
            search_container.pack(fill=tk.X, pady=(0, 8))

            tk.Label(
                search_container,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            responsible_search_entry = tk.Entry(
                search_container,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1E3A5F"
            )
            responsible_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث
            list_frame = tk.Frame(left_resp, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            list_frame.pack(fill=tk.BOTH, expand=True)

            scrollbar = tk.Scrollbar(list_frame, width=12)
            scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            responsible_listbox = tk.Listbox(
                list_frame,
                font=("Tajawal", 11),
                height=4,
                bd=0,
                highlightthickness=0,
                yscrollcommand=scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#E3F2FD",
                selectforeground="#0D47A1"
            )
            responsible_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            scrollbar.config(command=responsible_listbox.yview)

            # زر التعيين
            assign_resp_btn = tk.Button(
                left_resp,
                text="تعيين كرئيس",
                font=("Tajawal", 11, "bold"),
                bg="#3498DB",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            assign_resp_btn.pack(pady=(8, 0))

            # القسم الثاني: أعضاء هيئة التدريس - مقسم إلى قسمين
            members_container = tk.Frame(main_container, bg="#F5F5F5")
            members_container.pack(fill=tk.BOTH, expand=True)

            # القسم الأيمن - منسوبي المدينة
            city_staff_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            city_staff_frame.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True, padx=(5, 0))

            # رأس قسم منسوبي المدينة - لون أزرق داكن رسمي
            city_header = tk.Frame(city_staff_frame, bg="#1B4F72", height=40)
            city_header.pack(fill=tk.X)
            city_header.pack_propagate(False)

            city_header_content = tk.Frame(city_header, bg="#1B4F72")
            city_header_content.pack(expand=True)

            tk.Label(
                city_header_content,
                text="أعضاء هيئة التدريس - منسوبي المدينة",
                font=("Tajawal", 14, "bold"),
                bg="#1B4F72",
                fg="white"
            ).pack(side=tk.RIGHT)

            city_count_label = tk.Label(
                city_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#1B4F72",
                fg="white"
            )
            city_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم منسوبي المدينة
            city_content = tk.Frame(city_staff_frame, bg="#FFFFFF", padx=15, pady=15)
            city_content.pack(fill=tk.BOTH, expand=True)

            # البحث في منسوبي المدينة
            city_search_frame = tk.Frame(city_content, bg="#FFFFFF")
            city_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                city_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            city_search_entry = tk.Entry(
                city_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#1B4F72"
            )
            city_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث لمنسوبي المدينة
            city_search_list_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            city_search_scrollbar = tk.Scrollbar(city_search_list_frame, width=12)
            city_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_search_listbox = tk.Listbox(
                city_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D4E6F1",
                selectforeground="#1B4F72"
            )
            city_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_search_scrollbar.config(command=city_search_listbox.yview)

            # زر إضافة منسوب مدينة
            add_city_btn = tk.Button(
                city_content,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#2874A6",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_city_btn.pack(pady=(5, 10))

            # خط فاصل
            tk.Frame(city_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من منسوبي المدينة
            tk.Label(
                city_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#1B4F72"
            ).pack(anchor=tk.E, pady=(0, 10))

            city_members_frame = tk.Frame(city_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            city_members_frame.pack(fill=tk.BOTH, expand=True)

            city_members_scrollbar = tk.Scrollbar(city_members_frame, width=12)
            city_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            city_members_listbox = tk.Listbox(
                city_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=city_members_scrollbar.set,
                bg="#EBF5FB",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            city_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            city_members_scrollbar.config(command=city_members_listbox.yview)

            # زر حذف منسوب مدينة
            remove_city_btn = tk.Button(
                city_content,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_city_btn.pack(pady=(10, 0))

            # القسم الأيسر - المتعاونين
            collaborators_frame = tk.Frame(members_container, bg="#FFFFFF", bd=1, relief=tk.RIDGE)
            collaborators_frame.pack(side=tk.LEFT, fill=tk.BOTH, expand=True, padx=(0, 5))

            # رأس قسم المتعاونين - لون رمادي داكن رسمي
            collab_header = tk.Frame(collaborators_frame, bg="#424949", height=40)
            collab_header.pack(fill=tk.X)
            collab_header.pack_propagate(False)

            collab_header_content = tk.Frame(collab_header, bg="#424949")
            collab_header_content.pack(expand=True)

            tk.Label(
                collab_header_content,
                text="أعضاء هيئة التدريس - المتعاونين",
                font=("Tajawal", 14, "bold"),
                bg="#424949",
                fg="white"
            ).pack(side=tk.RIGHT)

            collab_count_label = tk.Label(
                collab_header_content,
                text="",
                font=("Tajawal", 12),
                bg="#424949",
                fg="white"
            )
            collab_count_label.pack(side=tk.RIGHT, padx=(15, 0))

            # محتوى قسم المتعاونين
            collab_content = tk.Frame(collaborators_frame, bg="#FFFFFF", padx=15, pady=15)
            collab_content.pack(fill=tk.BOTH, expand=True)

            # البحث في المتعاونين
            collab_search_frame = tk.Frame(collab_content, bg="#FFFFFF")
            collab_search_frame.pack(fill=tk.X, pady=(0, 10))

            tk.Label(
                collab_search_frame,
                text="البحث:",
                font=("Tajawal", 12),
                bg="#FFFFFF",
                fg="#666"
            ).pack(side=tk.RIGHT, padx=(0, 8))

            collab_search_entry = tk.Entry(
                collab_search_frame,
                font=("Tajawal", 12),
                bd=1,
                relief=tk.SOLID,
                highlightthickness=1,
                highlightcolor="#424949"
            )
            collab_search_entry.pack(side=tk.RIGHT, fill=tk.X, expand=True)

            # قائمة البحث للمتعاونين
            collab_search_list_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_search_list_frame.pack(fill=tk.BOTH, expand=True, pady=(0, 10))

            collab_search_scrollbar = tk.Scrollbar(collab_search_list_frame, width=12)
            collab_search_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_search_listbox = tk.Listbox(
                collab_search_list_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_search_scrollbar.set,
                bg="#FAFAFA",
                fg="#333333",
                selectbackground="#D5DBDB",
                selectforeground="#212F3C"
            )
            collab_search_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_search_scrollbar.config(command=collab_search_listbox.yview)

            # زر إضافة متعاون
            add_collab_btn = tk.Button(
                collab_content,
                text="إضافة العضو",
                font=("Tajawal", 11, "bold"),
                bg="#5D6D7E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            add_collab_btn.pack(pady=(5, 10))

            # خط فاصل
            tk.Frame(collab_content, bg="#E0E0E0", height=2).pack(fill=tk.X, pady=10)

            # الأعضاء الحاليون من المتعاونين
            tk.Label(
                collab_content,
                text="الأعضاء الحاليون:",
                font=("Tajawal", 12, "bold"),
                bg="#FFFFFF",
                fg="#424949"
            ).pack(anchor=tk.E, pady=(0, 10))

            collab_members_frame = tk.Frame(collab_content, bg="#FFFFFF", relief=tk.RIDGE, bd=1)
            collab_members_frame.pack(fill=tk.BOTH, expand=True)

            collab_members_scrollbar = tk.Scrollbar(collab_members_frame, width=12)
            collab_members_scrollbar.pack(side=tk.LEFT, fill=tk.Y)

            collab_members_listbox = tk.Listbox(
                collab_members_frame,
                font=("Tajawal", 11),
                height=6,
                bd=0,
                highlightthickness=0,
                yscrollcommand=collab_members_scrollbar.set,
                bg="#F4F6F6",
                fg="#333333",
                selectbackground="#FFEBEE",
                selectforeground="#B71C1C"
            )
            collab_members_listbox.pack(side=tk.RIGHT, fill=tk.BOTH, expand=True)
            collab_members_scrollbar.config(command=collab_members_listbox.yview)

            # زر حذف متعاون
            remove_collab_btn = tk.Button(
                collab_content,
                text="حذف العضو المحدد",
                font=("Tajawal", 11, "bold"),
                bg="#B03A2E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=15,
                pady=6
            )
            remove_collab_btn.pack(pady=(10, 0))

            # الشريط السفلي
            footer_frame = tk.Frame(manage_window, bg="#1E3A5F", height=50)
            footer_frame.pack(fill=tk.X)
            footer_frame.pack_propagate(False)

            close_btn = tk.Button(
                footer_frame,
                text="إغلاق",
                font=("Tajawal", 13, "bold"),
                bg="#34495E",
                fg="white",
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                padx=35,
                pady=10,
                command=lambda: [manage_window.destroy(), load_course_paths(search_entry.get())]
            )
            close_btn.place(relx=0.5, rely=0.5, anchor=tk.CENTER)

            # maps لتخزين معرفات المدرسين
            manage_window.city_members_map = {}
            manage_window.collab_members_map = {}
            manage_window.city_search_map = {}
            manage_window.collab_search_map = {}
            manage_window.responsible_map = {}

            # تحميل البيانات الحالية
            def load_current_data():
                cursor = self.db_conn.cursor()

                # تحميل المدرس المسؤول
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE ctp.course_name = ? AND ctp.is_responsible = 1
                """, (course_name,))

                responsible = cursor.fetchone()
                if responsible:
                    current_responsible_label.config(
                        text=f"{responsible[1]}\n{responsible[2]} - {responsible[3]}",
                        font=("Tajawal", 11, "bold")
                    )
                    remove_resp_btn.config(state=tk.NORMAL)
                else:
                    current_responsible_label.config(
                        text="لم يتم تعيين مسؤول بعد",
                        fg="#999999",
                        font=("Tajawal", 11, "italic")
                    )
                    remove_resp_btn.config(state=tk.DISABLED)

                # تحميل أعضاء هيئة التدريس - منسوبي المدينة
                city_members_listbox.delete(0, tk.END)
                manage_window.city_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                    AND t.category = 'منسوبي المدينة'
                    ORDER BY t.name
                """, (course_name,))

                city_members = cursor.fetchall()
                for i, (teacher_id, name, rank, workplace) in enumerate(city_members):
                    display_text = f"{name} - {rank} ({workplace})"
                    city_members_listbox.insert(tk.END, display_text)
                    manage_window.city_members_map[i] = teacher_id

                city_count_label.config(text=f"العدد: {len(city_members)}")

                # تحميل أعضاء هيئة التدريس - المتعاونين
                collab_members_listbox.delete(0, tk.END)
                manage_window.collab_members_map.clear()

                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    ORDER BY t.name
                """, (course_name,))

                collab_members = cursor.fetchall()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(collab_members):
                    display_text = f"{name} - {rank} ({workplace}) - {category}"
                    collab_members_listbox.insert(tk.END, display_text)
                    manage_window.collab_members_map[i] = teacher_id

                collab_count_label.config(text=f"العدد: {len(collab_members)}")

            # البحث عن المدرسين للمسؤول (يبحث في الجميع)
            def search_responsible_teachers(*args):
                search_term = responsible_search_entry.get().strip()
                responsible_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT id, name, rank, workplace, category
                    FROM teachers 
                    WHERE LOWER(name) LIKE LOWER(?)
                    ORDER BY name
                    LIMIT 20
                """, (f'%{search_term}%',))

                teachers = cursor.fetchall()
                manage_window.responsible_map.clear()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace})"
                    if category:
                        display_text += f" [{category}]"
                    responsible_listbox.insert(tk.END, display_text)
                    manage_window.responsible_map[i] = teacher_id

            # البحث في منسوبي المدينة
            def search_city_teachers(*args):
                search_term = city_search_entry.get().strip()
                city_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category = 'منسوبي المدينة'
                    AND t.id NOT IN (
                        SELECT teacher_id FROM course_teacher_paths 
                        WHERE course_name = ?
                    )
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%', course_name))

                teachers = cursor.fetchall()
                manage_window.city_search_map.clear()
                for i, (teacher_id, name, rank, workplace) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace})"
                    city_search_listbox.insert(tk.END, display_text)
                    manage_window.city_search_map[i] = teacher_id

            # البحث في المتعاونين
            def search_collab_teachers(*args):
                search_term = collab_search_entry.get().strip()
                collab_search_listbox.delete(0, tk.END)

                if len(search_term) < 1:
                    return

                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT t.id, t.name, t.rank, t.workplace, t.category
                    FROM teachers t
                    WHERE LOWER(t.name) LIKE LOWER(?)
                    AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                    AND t.id NOT IN (
                        SELECT teacher_id FROM course_teacher_paths 
                        WHERE course_name = ?
                    )
                    ORDER BY t.name
                    LIMIT 20
                """, (f'%{search_term}%', course_name))

                teachers = cursor.fetchall()
                manage_window.collab_search_map.clear()
                for i, (teacher_id, name, rank, workplace, category) in enumerate(teachers):
                    display_text = f"{name} - {rank} ({workplace}) - {category}"
                    collab_search_listbox.insert(tk.END, display_text)
                    manage_window.collab_search_map[i] = teacher_id

            # تعيين مدرس مسؤول
            def assign_responsible():
                selection = responsible_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.responsible_map[selection[0]]

                # التحقق من وجود المدرس في برامج أخرى
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT DISTINCT ctp.course_name
                    FROM course_teacher_paths ctp
                    WHERE ctp.teacher_id = ? AND ctp.course_name != ?
                """, (teacher_id, course_name))

                other_courses = cursor.fetchall()
                if other_courses:
                    courses_list = "\n".join([f"• {c[0]}" for c in other_courses])
                    messagebox.showerror(
                        "تعارض",
                        f"لا يمكن تعيين هذا المدرس كرئيس للبرنامج التدريبي\n\n"
                        f"المدرس عضو في البرامج التدريبية التالية:\n{courses_list}"
                    )
                    return

                if messagebox.askyesno("تأكيد", "هل تريد تعيين هذا المدرس كرئيس للبرنامج التدريبي؟"):
                    try:
                        cursor = self.db_conn.cursor()

                        # إلغاء أي مسؤول سابق
                        cursor.execute("""
                            UPDATE course_teacher_paths 
                            SET is_responsible = 0 
                            WHERE course_name = ? AND is_responsible = 1
                        """, (course_name,))

                        # تعيين المسؤول الجديد
                        cursor.execute("""
                            INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                            VALUES (?, ?, 1, ?)
                            ON CONFLICT(course_name, teacher_id) 
                            DO UPDATE SET is_responsible = 1
                        """, (course_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                        self.db_conn.commit()

                        load_current_data()
                        responsible_search_entry.delete(0, tk.END)
                        responsible_listbox.delete(0, tk.END)

                        messagebox.showinfo("نجاح", "تم تعيين رئيس البرنامج التدريبي بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف المسؤول
            def remove_responsible():
                if messagebox.askyesno("تأكيد", "هل تريد حذف المسؤول الحالي؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM course_teacher_paths 
                            WHERE course_name = ? AND is_responsible = 1
                        """, (course_name,))

                        self.db_conn.commit()
                        load_current_data()
                        messagebox.showinfo("نجاح", "تم حذف المسؤول بنجاح")

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة منسوب مدينة
            def add_city_member():
                selection = city_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.city_search_map[selection[0]]

                # التحقق من وجود المدرس في برامج أخرى
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT DISTINCT ctp.course_name
                    FROM course_teacher_paths ctp
                    WHERE ctp.teacher_id = ? AND ctp.course_name != ?
                """, (teacher_id, course_name))

                other_courses = cursor.fetchall()
                if other_courses:
                    courses_list = "\n".join([f"• {c[0]}" for c in other_courses])
                    messagebox.showerror(
                        "تعارض",
                        f"لا يمكن إضافة هذا المدرس كعضو في البرنامج التدريبي\n\n"
                        f"المدرس عضو في البرامج التدريبية التالية:\n{courses_list}"
                    )
                    return

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (course_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    city_search_entry.delete(0, tk.END)
                    city_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # إضافة متعاون
            def add_collab_member():
                selection = collab_search_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار مدرس من القائمة")
                    return

                teacher_id = manage_window.collab_search_map[selection[0]]

                # التحقق من وجود المدرس في برامج أخرى
                cursor = self.db_conn.cursor()
                cursor.execute("""
                    SELECT DISTINCT ctp.course_name
                    FROM course_teacher_paths ctp
                    WHERE ctp.teacher_id = ? AND ctp.course_name != ?
                """, (teacher_id, course_name))

                other_courses = cursor.fetchall()
                if other_courses:
                    courses_list = "\n".join([f"• {c[0]}" for c in other_courses])
                    messagebox.showerror(
                        "تعارض",
                        f"لا يمكن إضافة هذا المدرس كعضو في البرنامج التدريبي\n\n"
                        f"المدرس عضو في البرامج التدريبية التالية:\n{courses_list}"
                    )
                    return

                try:
                    cursor = self.db_conn.cursor()

                    cursor.execute("""
                        INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (course_name, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    self.db_conn.commit()

                    load_current_data()
                    collab_search_entry.delete(0, tk.END)
                    collab_search_listbox.delete(0, tk.END)

                    messagebox.showinfo("نجاح", "تمت إضافة العضو بنجاح")

                except Exception as e:
                    messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف منسوب مدينة
            def remove_city_member():
                selection = city_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو للحذف")
                    return

                teacher_id = manage_window.city_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من هيئة التدريس؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM course_teacher_paths 
                            WHERE course_name = ? AND teacher_id = ? AND is_responsible = 0
                        """, (course_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # حذف متعاون
            def remove_collab_member():
                selection = collab_members_listbox.curselection()
                if not selection:
                    messagebox.showwarning("تنبيه", "يرجى اختيار عضو للحذف")
                    return

                teacher_id = manage_window.collab_members_map[selection[0]]

                if messagebox.askyesno("تأكيد", "هل تريد حذف هذا العضو من هيئة التدريس؟"):
                    try:
                        cursor = self.db_conn.cursor()
                        cursor.execute("""
                            DELETE FROM course_teacher_paths 
                            WHERE course_name = ? AND teacher_id = ? AND is_responsible = 0
                        """, (course_name, teacher_id))

                        self.db_conn.commit()
                        load_current_data()

                    except Exception as e:
                        messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

            # ربط الأحداث
            responsible_search_entry.bind('<KeyRelease>', search_responsible_teachers)
            city_search_entry.bind('<KeyRelease>', search_city_teachers)
            collab_search_entry.bind('<KeyRelease>', search_collab_teachers)

            assign_resp_btn.config(command=assign_responsible)
            remove_resp_btn.config(command=remove_responsible)
            add_city_btn.config(command=add_city_member)
            add_collab_btn.config(command=add_collab_member)
            remove_city_btn.config(command=remove_city_member)
            remove_collab_btn.config(command=remove_collab_member)

            # تحميل البيانات الأولية
            load_current_data()

        # دالة تصدير البيانات
        def export_faculty_data():
            """تصدير بيانات هيئة التدريس لجميع الدورات"""
            if not EXCEL_AVAILABLE:
                messagebox.showerror("خطأ", "يجب تثبيت المكتبات المطلوبة\npip install pandas openpyxl")
                return

            try:
                from tkinter import filedialog
                import pandas as pd
                from openpyxl import Workbook
                from openpyxl.styles import Alignment, Font, PatternFill, Border, Side
                from openpyxl.utils import get_column_letter

                # اختيار مكان الحفظ
                file_path = filedialog.asksaveasfilename(
                    title="حفظ بيانات هيئة التدريس",
                    defaultextension=".xlsx",
                    initialfile=f"هيئة_التدريس_{datetime.now().strftime('%Y%m%d_%H%M%S')}.xlsx",
                    filetypes=[("Excel files", "*.xlsx"), ("All files", "*.*")]
                )

                if not file_path:
                    return

                cursor = self.db_conn.cursor()

                # الحصول على جميع الدورات
                cursor.execute("""
                    SELECT DISTINCT course_name 
                    FROM course_teacher_paths 
                    ORDER BY course_name
                """)
                courses = cursor.fetchall()

                # إنشاء ملف Excel
                wb = Workbook()
                ws = wb.active
                ws.title = "هيئة التدريس"

                # تعيين اتجاه الورقة من اليمين لليسار
                ws.sheet_view.rightToLeft = True

                # العناوين
                headers = ["م", "اسم الدورة", "رئيس البرنامج التدريبي", "منسوبي المدينة", "المتعاونين", "العدد الكلي"]

                # كتابة العناوين
                for col, header in enumerate(headers, 1):
                    cell = ws.cell(row=1, column=col, value=header)
                    cell.font = Font(name='Arial', size=14, bold=True, color="FFFFFF")
                    cell.alignment = Alignment(horizontal='center', vertical='center', wrap_text=True)
                    cell.fill = PatternFill(start_color="1E3A5F", end_color="1E3A5F", fill_type="solid")

                # تنسيق الحدود
                thin_border = Border(
                    left=Side(style='thin'),
                    right=Side(style='thin'),
                    top=Side(style='thin'),
                    bottom=Side(style='thin')
                )

                # كتابة البيانات
                row_num = 2
                for idx, (course_name,) in enumerate(courses, 1):
                    # المسؤول
                    cursor.execute("""
                        SELECT t.name, t.rank 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 1
                    """, (course_name,))
                    responsible = cursor.fetchone()
                    responsible_text = f"{responsible[0]} - {responsible[1]}" if responsible else "غير محدد"

                    # منسوبي المدينة
                    cursor.execute("""
                        SELECT t.name, t.rank 
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category = 'منسوبي المدينة'
                        ORDER BY t.name
                    """, (course_name,))
                    city_staff = cursor.fetchall()
                    city_staff_text = "\n".join([f"{m[0]} - {m[1]}" for m in city_staff]) if city_staff else "لا يوجد"

                    # المتعاونين
                    cursor.execute("""
                        SELECT t.name, t.rank, t.category
                        FROM course_teacher_paths ctp
                        JOIN teachers t ON ctp.teacher_id = t.id
                        WHERE ctp.course_name = ? AND ctp.is_responsible = 0
                        AND t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY t.name
                    """, (course_name,))
                    collaborators = cursor.fetchall()
                    collab_text = "\n".join(
                        [f"{c[0]} - {c[1]} ({c[2]})" for c in collaborators]) if collaborators else "لا يوجد"

                    # العدد الكلي
                    total = len(city_staff) + len(collaborators) + (1 if responsible else 0)

                    # كتابة الصف
                    data_row = [idx, course_name, responsible_text, city_staff_text, collab_text, total]
                    for col, value in enumerate(data_row, 1):
                        cell = ws.cell(row=row_num, column=col, value=value)
                        cell.font = Font(name='Arial', size=12)
                        cell.alignment = Alignment(
                            horizontal='right' if col > 1 else 'center',
                            vertical='center',
                            wrap_text=True
                        )
                        cell.border = thin_border

                        # تلوين الصفوف بالتناوب
                        if row_num % 2 == 0:
                            cell.fill = PatternFill(start_color="F0F0F0", end_color="F0F0F0", fill_type="solid")

                    row_num += 1

                # تعيين عرض الأعمدة
                ws.column_dimensions['A'].width = 8  # م
                ws.column_dimensions['B'].width = 40  # اسم الدورة
                ws.column_dimensions['C'].width = 35  # المسؤول
                ws.column_dimensions['D'].width = 50  # منسوبي المدينة
                ws.column_dimensions['E'].width = 50  # المتعاونين
                ws.column_dimensions['F'].width = 15  # العدد الكلي

                # إضافة ورقة الملخص
                summary_ws = wb.create_sheet("الملخص")
                summary_ws.sheet_view.rightToLeft = True

                # ملخص البيانات
                summary_ws['A1'] = "ملخص بيانات هيئة التدريس"
                summary_ws['A1'].font = Font(name='Arial', size=16, bold=True)
                summary_ws.merge_cells('A1:B1')

                # حساب الإحصائيات
                cursor.execute("""
                    SELECT COUNT(DISTINCT teacher_id) 
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE t.category = 'منسوبي المدينة'
                """)
                city_staff_count = cursor.fetchone()[0]

                cursor.execute("""
                    SELECT COUNT(DISTINCT teacher_id) 
                    FROM course_teacher_paths ctp
                    JOIN teachers t ON ctp.teacher_id = t.id
                    WHERE t.category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                """)
                collab_count = cursor.fetchone()[0]

                summary_data = [
                    ["إجمالي الدورات", len(courses)],
                    ["إجمالي منسوبي المدينة", city_staff_count],
                    ["إجمالي المتعاونين", collab_count],
                    ["تاريخ التصدير", datetime.now().strftime('%Y-%m-%d')],
                    ["وقت التصدير", datetime.now().strftime('%H:%M:%S')]
                ]

                for idx, (label, value) in enumerate(summary_data, 3):
                    summary_ws[f'A{idx}'] = label
                    summary_ws[f'B{idx}'] = value
                    summary_ws[f'A{idx}'].font = Font(name='Arial', size=12, bold=True)
                    summary_ws[f'B{idx}'].font = Font(name='Arial', size=12)

                # حفظ الملف
                wb.save(file_path)
                messagebox.showinfo("نجاح", f"تم تصدير البيانات بنجاح\n{file_path}")

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ أثناء التصدير:\n{str(e)}")

        # ربط الأزرار
        add_course_btn.config(command=add_course_to_paths)
        delete_course_btn.config(command=delete_course_from_paths)
        export_btn.config(command=export_faculty_data)

        # ربط النقر على الجدول
        paths_tree.bind("<Double-Button-1>", manage_course_teachers)

        # البحث الديناميكي
        def dynamic_search(*args):
            search_term = search_entry.get().strip()
            load_course_paths(search_term)

        # ربط البحث الديناميكي
        search_entry.bind('<KeyRelease>', dynamic_search)

        # تسمية للمساعدة
        help_label = tk.Label(
            paths_window,
            text="💡 انقر مرتين على أي دورة لإدارة هيئة التدريس",
            font=("Tajawal", 12),
            bg=self.COLORS["background"],
            fg="#666"
        )
        help_label.pack(pady=5)

        # زر الإغلاق
        close_btn = tk.Button(
            paths_window,
            text="إغلاق",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["dark"],
            fg="white",
            bd=0,
            padx=30,
            pady=12,
            cursor="hand2",
            command=paths_window.destroy
        )
        close_btn.pack(pady=10)

        # تحميل البيانات
        load_course_paths()

    def _search_teachers(self):
        """البحث الديناميكي في المدرسين"""
        # الحصول على نص البحث
        search_term = self.search_var.get().strip()
        filter_category = self.filter_var.get()

        # مسح جميع العناصر
        for item in self.teachers_tree.get_children():
            self.teachers_tree.delete(item)

        try:
            cursor = self.db_conn.cursor()

            # بناء استعلام SQL
            if filter_category == "الكل":
                if search_term:
                    # البحث في الاسم ورقم الهوية
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE name LIKE ? OR id_number LIKE ?
                        ORDER BY name
                    """, (f'%{search_term}%', f'%{search_term}%'))
                else:
                    # عرض الكل
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        ORDER BY name
                    """)
            elif filter_category == "منسوبي المدينة":
                if search_term:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category = 'منسوبي المدينة' 
                        AND (name LIKE ? OR id_number LIKE ?)
                        ORDER BY name
                    """, (f'%{search_term}%', f'%{search_term}%'))
                else:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category = 'منسوبي المدينة'
                        ORDER BY name
                    """)
            elif filter_category == "متعاونين":
                if search_term:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category IN ('متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد') 
                        AND (name LIKE ? OR id_number LIKE ?)
                        ORDER BY name
                    """, (f'%{search_term}%', f'%{search_term}%'))
                else:
                    cursor.execute("""
                        SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                        FROM teachers 
                        WHERE category IN ('متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
                        ORDER BY name
                    """)

            teachers = cursor.fetchall()

            # إضافة النتائج إلى الشجرة
            for index, teacher in enumerate(teachers):
                display_data = (
                    teacher[1],  # name
                    teacher[2],  # rank
                    teacher[3],  # id_number
                    teacher[4],  # workplace
                    teacher[5],  # qualification
                    teacher[6]  # phone
                )

                tag = 'evenrow' if index % 2 == 0 else 'oddrow'
                item = self.teachers_tree.insert("", tk.END, values=display_data, tags=(tag, f"id_{teacher[0]}"))

            # تحديث العدادات
            self._update_teachers_count()

            # تحديث نص زر التصدير
            if filter_category != "الكل":
                self.export_btn.config(text=f"📊 تصدير بيانات {filter_category}")
            else:
                self.export_btn.config(text="📊 تصدير جميع البيانات")

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في البحث: {str(e)}")
            print(f"تفاصيل الخطأ: {str(e)}")  # للتشخيص

    def _filter_teachers(self):
        """تصفية المدرسين حسب الفئة"""
        self._search_teachers()  # استخدام نفس دالة البحث

    def _update_teachers_count(self):
        """تحديث عدد المدرسين مع إظهار التفاصيل"""
        try:
            cursor = self.db_conn.cursor()

            # عدد منسوبي المدينة
            cursor.execute("SELECT COUNT(*) FROM teachers WHERE category = 'منسوبي المدينة'")
            city_count = cursor.fetchone()[0]

            # عدد المتعاونين (جميع الفئات بما فيها "متعاون" فقط)
            cursor.execute("""
                SELECT COUNT(*) FROM teachers 
                WHERE category IN ('متعاون', 'متعاون مدني', 'متعاون عسكري', 'متعاون عسكري متقاعد')
            """)
            collaborator_count = cursor.fetchone()[0]

            # العدد الإجمالي
            total_count = city_count + collaborator_count

            # تحديث النص
            text = f"إجمالي المدرسين: {total_count} | منسوبي المدينة: {city_count} | المتعاونين: {collaborator_count}"

            self.teacher_count_label.config(text=text)

        except Exception as e:
            print(f"خطأ في تحديث العدادات: {e}")

        finally:
            cursor.close()

    def _export_filtered_data(self):
        """تصدير البيانات المفلترة إلى Excel"""
        if not EXCEL_AVAILABLE:
            messagebox.showerror("خطأ", "يجب تثبيت المكتبات المطلوبة\npip install pandas openpyxl")
            return

        try:
            from tkinter import filedialog

            # الحصول على البيانات المعروضة حالياً
            data = []
            for item in self.teachers_tree.get_children():
                values = self.teachers_tree.item(item)['values']
                data.append(values)

            if not data:
                messagebox.showinfo("تنبيه", "لا توجد بيانات للتصدير")
                return

            # تحديد اسم الملف
            filter_category = self.filter_combo.get() if hasattr(self, 'filter_combo') else "الكل"
            default_name = f"مدرسين_{filter_category}" if filter_category != "الكل" else "جميع_المدرسين"

            file_path = filedialog.asksaveasfilename(
                title="حفظ ملف Excel",
                defaultextension=".xlsx",
                initialfile=f"{default_name}_{datetime.now().strftime('%Y%m%d_%H%M%S')}.xlsx",
                filetypes=[("Excel files", "*.xlsx"), ("All files", "*.*")]
            )

            if not file_path:
                return

            # إنشاء DataFrame
            columns = ["الاسم", "الرتبة", "رقم الهوية", "جهة العمل", "المؤهل الدراسي", "رقم الجوال"]
            df = pd.DataFrame(data, columns=columns)

            # إنشاء ملف Excel مع التنسيق
            with pd.ExcelWriter(file_path, engine='openpyxl') as writer:
                # كتابة البيانات
                df.to_excel(writer, sheet_name='المدرسون', index=False)

                # الحصول على ورقة العمل
                worksheet = writer.sheets['المدرسون']

                # تعيين اتجاه الورقة من اليمين لليسار
                worksheet.sheet_view.rightToLeft = True

                # تنسيق بسيط
                thin_border = Border(
                    left=Side(style='thin'),
                    right=Side(style='thin'),
                    top=Side(style='thin'),
                    bottom=Side(style='thin')
                )

                # تنسيق رؤوس الأعمدة
                for col_num, column in enumerate(columns, 1):
                    cell = worksheet.cell(row=1, column=col_num)
                    cell.font = Font(name='Arial', size=12, bold=True)
                    cell.alignment = Alignment(
                        horizontal='center',
                        vertical='center',
                        wrap_text=True
                    )
                    cell.border = thin_border

                # تنسيق البيانات
                for row_num in range(2, len(df) + 2):
                    for col_num in range(1, len(columns) + 1):
                        cell = worksheet.cell(row=row_num, column=col_num)
                        cell.font = Font(name='Arial', size=11)
                        cell.alignment = Alignment(
                            horizontal='right' if col_num != 3 else 'center',
                            vertical='center',
                            wrap_text=True
                        )
                        cell.border = thin_border

                # تعيين عرض الأعمدة
                column_widths = {
                    'A': 35,  # الاسم
                    'B': 20,  # الرتبة
                    'C': 15,  # رقم الهوية
                    'D': 35,  # جهة العمل
                    'E': 25,  # المؤهل
                    'F': 15  # رقم الجوال
                }

                for column, width in column_widths.items():
                    worksheet.column_dimensions[column].width = width

                # إضافة ورقة الملخص
                summary_data = {
                    'الفئة': [filter_category if filter_category != "الكل" else "جميع الفئات"],
                    'عدد المدرسين': [len(data)],
                    'تاريخ التصدير': [datetime.now().strftime('%Y-%m-%d')],
                    'وقت التصدير': [datetime.now().strftime('%H:%M:%S')],
                    'المستخدم': [self.current_user['full_name'] if self.current_user else 'غير محدد']
                }

                summary_df = pd.DataFrame(summary_data)
                summary_df.to_excel(writer, sheet_name='ملخص', index=False)

            messagebox.showinfo("نجاح", f"تم تصدير البيانات بنجاح\n{file_path}")

        except Exception as e:
            messagebox.showerror("خطأ", f"حدث خطأ أثناء التصدير:\n{str(e)}")

    def _load_teachers(self):
        """تحميل بيانات المدرسين من قاعدة البيانات مع التنسيق المحسن"""
        # إعادة تعيين البحث والفلتر
        if hasattr(self, 'search_var'):
            self.search_var.set("")
        if hasattr(self, 'filter_var'):
            self.filter_var.set("الكل")

        # مسح البيانات الحالية
        for item in self.teachers_tree.get_children():
            self.teachers_tree.delete(item)

        try:
            cursor = self.db_conn.cursor()
            cursor.execute("""
                SELECT id, name, rank, id_number, workplace, qualification, phone, category 
                FROM teachers 
                ORDER BY name
            """)

            teachers = cursor.fetchall()

            # إضافة البيانات مع ألوان متناوبة
            for index, teacher in enumerate(teachers):
                # البيانات المعروضة (بدون ID)
                display_data = (
                    teacher[1],  # name
                    teacher[2],  # rank
                    teacher[3],  # id_number
                    teacher[4],  # workplace
                    teacher[5],  # qualification
                    teacher[6]  # phone
                )

                tag = 'evenrow' if index % 2 == 0 else 'oddrow'
                item = self.teachers_tree.insert("", tk.END, values=display_data, tags=(tag, f"id_{teacher[0]}"))

            # تحديث عدادات المدرسين
            self._update_teachers_count()

            # تحديث نص زر التصدير
            if hasattr(self, 'export_btn'):
                self.export_btn.config(text="📊 تصدير جميع البيانات")

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في تحميل البيانات: {str(e)}")

    def _check_categories(self):
        """التحقق من الفئات الموجودة في قاعدة البيانات"""
        try:
            cursor = self.db_conn.cursor()
            cursor.execute("SELECT DISTINCT category FROM teachers WHERE category IS NOT NULL")
            categories = cursor.fetchall()

            print("الفئات الموجودة في قاعدة البيانات:")
            for cat in categories:
                print(f"- '{cat[0]}'")

            # يمكنك استدعاء هذه الدالة مؤقتاً للتحقق من الفئات

        except Exception as e:
            print(f"خطأ: {str(e)}")

    def _fix_categories(self):
        """إصلاح أسماء الفئات في قاعدة البيانات"""
        try:
            cursor = self.db_conn.cursor()

            # تحديث الفئات لتتطابق مع القائمة المنسدلة
            updates = [
                ("متعاون", "متعاون مدني"),  # إذا كانت مخزنة كـ "متعاون" فقط
                ("متعاون عسكري متقاعد", "متعاون عسكري متقاعد"),  # للتأكد من عدم وجود مسافات زائدة
            ]

            for old_name, new_name in updates:
                cursor.execute("UPDATE teachers SET category = ? WHERE category = ? OR category LIKE ?",
                               (new_name, old_name, f"%{old_name}%"))

            self.db_conn.commit()
            messagebox.showinfo("نجاح", "تم تحديث الفئات")
            self._load_teachers()

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في تحديث الفئات: {str(e)}")

    # 1. دالة إضافة مدرس جديد كاملة
    def _open_add_teacher_window(self):
        """فتح نافذة إضافة مدرس جديد مع مسار التخصص بدلاً من الدورات"""
        add_window = tk.Toplevel(self)
        add_window.title("إضافة مدرس جديد")
        add_window.geometry("700x750")
        add_window.configure(bg=self.COLORS["background"])
        add_window.transient(self)
        add_window.grab_set()

        # توسيط النافذة
        add_window.update_idletasks()
        x = (add_window.winfo_screenwidth() - 700) // 2
        y = (add_window.winfo_screenheight() - 750) // 2
        add_window.geometry(f"700x750+{x}+{y}")

        # شريط العنوان
        header_frame = tk.Frame(add_window, bg="#1E3A5F", height=80)
        header_frame.pack(fill=tk.X)
        header_frame.pack_propagate(False)

        title_label = tk.Label(
            header_frame,
            text="إضافة مدرس جديد",
            font=("Tajawal", 24, "bold"),
            bg="#1E3A5F",
            fg="white"
        )
        title_label.pack(expand=True)

        # إطار المحتوى الرئيسي
        main_frame = tk.Frame(add_window, bg=self.COLORS["background"])
        main_frame.pack(fill=tk.BOTH, expand=True, padx=20, pady=20)

        # إطار الحقول
        form_frame = tk.Frame(main_frame, bg="#FFFFFF", bd=2, relief=tk.RIDGE)
        form_frame.pack(fill=tk.BOTH, expand=True)

        inner_form = tk.Frame(form_frame, bg="#FFFFFF", padx=40, pady=30)
        inner_form.pack(fill=tk.BOTH, expand=True)

        # تنسيق الحقول
        label_font = ("Tajawal", 14, "bold")
        entry_font = ("Tajawal", 13)

        # حقل الاسم
        tk.Label(inner_form, text="الاسم الكامل *", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=0, column=0, sticky=tk.W, pady=15)
        name_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                              highlightthickness=2, highlightcolor="#1E3A5F")
        name_entry.grid(row=0, column=1, pady=15, padx=10)

        # حقل الرتبة
        tk.Label(inner_form, text="الرتبة *", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=1, column=0, sticky=tk.W, pady=15)
        rank_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                              highlightthickness=2, highlightcolor="#1E3A5F")
        rank_entry.grid(row=1, column=1, pady=15, padx=10)

        # حقل رقم الهوية
        tk.Label(inner_form, text="رقم الهوية * (10 أرقام)", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=2, column=0, sticky=tk.W, pady=15)
        id_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                            highlightthickness=2, highlightcolor="#1E3A5F")
        id_entry.grid(row=2, column=1, pady=15, padx=10)

        # التحقق من إدخال الأرقام فقط
        def validate_id(char):
            return char.isdigit() or char == ""

        vcmd = (add_window.register(validate_id), '%S')
        id_entry.config(validate='key', validatecommand=vcmd)

        # حقل رقم الجوال
        tk.Label(inner_form, text="رقم الجوال (اختياري)", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=3, column=0, sticky=tk.W, pady=15)
        phone_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                               highlightthickness=2, highlightcolor="#1E3A5F")
        phone_entry.grid(row=3, column=1, pady=15, padx=10)

        # حقل جهة العمل
        tk.Label(inner_form, text="جهة العمل *", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=4, column=0, sticky=tk.W, pady=15)
        workplace_entry = tk.Entry(inner_form, font=entry_font, width=35, bd=2, relief=tk.FLAT,
                                   highlightthickness=2, highlightcolor="#1E3A5F")
        workplace_entry.grid(row=4, column=1, pady=15, padx=10)

        # حقل المؤهل الدراسي - محدث بالقيم الجديدة
        tk.Label(inner_form, text="المؤهل الدراسي *", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=5, column=0, sticky=tk.W, pady=15)

        qualifications = ["دكتوراه", "ماجستير", "دبلوم عالي", "بكالوريوس", "جامعة متوسطة", "ثانوي", "لا يوجد مؤهل"]
        qualification_var = tk.StringVar(master=add_window, value=qualifications[3])  # بكالوريوس كقيمة افتراضية

        # تنسيق Combobox
        style = ttk.Style()
        style.configure("Custom.TCombobox",
                        fieldbackground="white",
                        borderwidth=2,
                        relief="flat",
                        font=entry_font)

        qualification_combo = ttk.Combobox(inner_form, textvariable=qualification_var,
                                           values=qualifications, font=entry_font,
                                           width=33, state="readonly", style="Custom.TCombobox")
        qualification_combo.grid(row=5, column=1, pady=15, padx=10)

        # حقل فئة المدرس
        tk.Label(inner_form, text="فئة المدرس *", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=6, column=0, sticky=tk.W, pady=15)

        categories = ["منسوبي المدينة", "متعاون مدني", "متعاون عسكري", "متعاون عسكري متقاعد"]
        category_var = tk.StringVar(master=add_window, value=categories[0])
        category_combo = ttk.Combobox(inner_form, textvariable=category_var,
                                      values=categories, font=entry_font,
                                      width=33, state="readonly", style="Custom.TCombobox")
        category_combo.grid(row=6, column=1, pady=15, padx=10)

        # حقل مسار الدورة المتخصص فيها (بدلاً من الدورات الحاصل عليها)
        tk.Label(inner_form, text="مسار الدورة المتخصص فيها", font=label_font,
                 bg="#FFFFFF", fg="#1E3A5F").grid(row=7, column=0, sticky=tk.W, pady=15)

        specialization_frame = tk.Frame(inner_form, bg="#FFFFFF")
        specialization_frame.grid(row=7, column=1, pady=15, padx=10, sticky=tk.W)

        # الحصول على مسميات الدورات من قاعدة البيانات
        cursor = self.db_conn.cursor()
        cursor.execute("SELECT id, name FROM course_names WHERE is_active = 1 ORDER BY name")
        courses = cursor.fetchall()
        course_names = ["لا يوجد"] + [course[1] for course in courses] if courses else ["لا يوجد"]

        specialization_var = tk.StringVar(master=add_window, value="لا يوجد")
        specialization_combo = ttk.Combobox(specialization_frame, textvariable=specialization_var,
                                            values=course_names, font=entry_font,
                                            width=33, state="readonly", style="Custom.TCombobox")
        specialization_combo.pack()

        # إطار الأزرار السفلي
        buttons_frame = tk.Frame(add_window, bg=self.COLORS["background"])
        buttons_frame.pack(fill=tk.X, pady=20)

        def save_teacher():
            # التحقق من البيانات المطلوبة
            name = name_entry.get().strip()
            rank = rank_entry.get().strip()
            id_number = id_entry.get().strip()
            phone = phone_entry.get().strip()
            workplace = workplace_entry.get().strip()
            qualification = qualification_var.get()
            category = category_var.get()
            specialization = specialization_var.get()

            if not all([name, rank, id_number, workplace, qualification, category]):
                messagebox.showwarning("تنبيه", "يرجى ملء جميع الحقول المطلوبة")
                return

            if len(id_number) != 10:
                messagebox.showwarning("تنبيه", "رقم الهوية يجب أن يكون 10 أرقام")
                return

            try:
                cursor = self.db_conn.cursor()

                # التحقق من عدم تكرار رقم الهوية
                cursor.execute("SELECT COUNT(*) FROM teachers WHERE id_number = ?", (id_number,))
                if cursor.fetchone()[0] > 0:
                    messagebox.showerror("خطأ", "رقم الهوية مسجل مسبقاً")
                    return

                # إدراج المدرس الجديد
                cursor.execute("""
                    INSERT INTO teachers (name, rank, id_number, phone, workplace, 
                                        qualification, category, created_date)
                    VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                """, (name, rank, id_number, phone, workplace, qualification, category,
                      datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                teacher_id = cursor.lastrowid

                # إضافة مسار التخصص إذا كان محدداً
                if specialization != "لا يوجد":
                    cursor.execute("""
                        INSERT INTO course_teacher_paths (course_name, teacher_id, is_responsible, created_date)
                        VALUES (?, ?, 0, ?)
                    """, (specialization, teacher_id, datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                self.db_conn.commit()
                messagebox.showinfo("نجاح", "تم إضافة المدرس بنجاح")
                add_window.destroy()
                self._load_teachers()

            except Exception as e:
                messagebox.showerror("خطأ", f"حدث خطأ: {str(e)}")

        # زر الحفظ
        save_btn = tk.Button(
            buttons_frame,
            text="💾 حفظ البيانات",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["success"],
            fg="white",
            padx=30,
            pady=12,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=save_teacher
        )
        save_btn.pack(side=tk.LEFT, padx=20)

        # تأثيرات hover
        save_btn.bind("<Enter>", lambda e: save_btn.config(bg="#218838"))
        save_btn.bind("<Leave>", lambda e: save_btn.config(bg=self.COLORS["success"]))

        # زر الإلغاء
        cancel_btn = tk.Button(
            buttons_frame,
            text="❌ إلغاء",
            font=("Tajawal", 14, "bold"),
            bg=self.COLORS["danger"],
            fg="white",
            padx=30,
            pady=12,
            bd=0,
            relief=tk.FLAT,
            cursor="hand2",
            command=add_window.destroy
        )
        cancel_btn.pack(side=tk.RIGHT, padx=20)

        # تأثيرات hover
        cancel_btn.bind("<Enter>", lambda e: cancel_btn.config(bg="#c82333"))
        cancel_btn.bind("<Leave>", lambda e: cancel_btn.config(bg=self.COLORS["danger"]))

    def _import_teachers(self):
        """عرض خيارات استيراد المدرسين"""
        import_menu = tk.Menu(self, tearoff=0, font=self.FONTS["text"])
        import_menu.configure(bg="white", fg="#424242", activebackground="#e3f2fd")

        import_menu.add_command(
            label="استيراد مدرسين منسوبي المدينة",
            command=lambda: self._import_from_excel("منسوبي المدينة")
        )
        import_menu.add_command(
            label="استيراد مدرسين متعاونين",
            command=lambda: self._import_from_excel("متعاون")
        )

        # الحصول على موقع المؤشر
        x = self.winfo_pointerx()
        y = self.winfo_pointery()
        import_menu.tk_popup(x, y)

    def _apply_filters(self):
        """تطبيق البحث والفلتر على البيانات"""
        # الحصول على قيم البحث والفلتر
        search_text = self.search_entry.get().strip() if hasattr(self, 'search_entry') else ""
        filter_value = self.filter_combo.get() if hasattr(self, 'filter_combo') else "الكل"

        # إذا كان البحث قصير جداً، لا تبحث
        if len(search_text) == 1:
            return

        # مسح الجدول
        for item in self.teachers_tree.get_children():
            self.teachers_tree.delete(item)

        try:
            cursor = self.db_conn.cursor()

            # بناء الاستعلام
            query = "SELECT id, name, rank, id_number, workplace, qualification, phone, category FROM teachers WHERE 1=1"
            params = []

            # إضافة شرط البحث
            if search_text:
                query += " AND (name LIKE ? OR id_number LIKE ?)"
                params.extend([f'%{search_text}%', f'%{search_text}%'])

            # إضافة شرط الفلتر
            if filter_value == "منسوبي المدينة":
                query += " AND category = ?"
                params.append("منسوبي المدينة")
            elif filter_value == "متعاونين":
                # تشمل جميع أنواع المتعاونين
                query += " AND category IN (?, ?, ?, ?)"
                params.extend(["متعاون", "متعاون مدني", "متعاون عسكري", "متعاون عسكري متقاعد"])

            query += " ORDER BY name LIMIT 500"

            # تنفيذ الاستعلام
            cursor.execute(query, params)
            teachers = cursor.fetchall()

            # إضافة النتائج للجدول
            for index, teacher in enumerate(teachers):
                display_data = (
                    teacher[1],  # name
                    teacher[2],  # rank
                    teacher[3],  # id_number
                    teacher[4],  # workplace
                    teacher[5],  # qualification
                    teacher[6]  # phone
                )

                tag = 'evenrow' if index % 2 == 0 else 'oddrow'
                self.teachers_tree.insert("", tk.END, values=display_data, tags=(tag, f"id_{teacher[0]}"))

            # تحديث العدادات
            self._update_teachers_count()

            # تحديث زر التصدير
            if filter_value != "الكل":
                self.export_btn.config(text=f"📊 تصدير بيانات {filter_value}")
            else:
                self.export_btn.config(text="📊 تصدير جميع البيانات")

        except Exception as e:
            messagebox.showerror("خطأ", f"خطأ في تطبيق الفلاتر: {str(e)}")

        finally:
            cursor.close()

    def _select_collaborator_type(self):
        """نافذة لاختيار نوع المتعاون"""
        dialog = tk.Toplevel(self)
        dialog.title("اختر نوع المتعاون")
        dialog.geometry("400x350")  # زيادة الارتفاع قليلاً
        dialog.configure(bg=self.COLORS["background"])
        dialog.transient(self)
        dialog.grab_set()

        # توسيط النافذة
        dialog.update_idletasks()
        x = (dialog.winfo_screenwidth() - 400) // 2
        y = (dialog.winfo_screenheight() - 350) // 2
        dialog.geometry(f"400x350+{x}+{y}")

        # المتغير لحفظ الاختيار
        selected_type = tk.StringVar(master=dialog)

        # العنوان
        tk.Label(
            dialog,
            text="اختر نوع المتعاون",
            font=self.FONTS["title"],
            bg=self.COLORS["primary"],
            fg="white",
            pady=15
        ).pack(fill=tk.X)

        # إطار الخيارات
        options_frame = tk.Frame(dialog, bg=self.COLORS["surface"], padx=20, pady=20)
        options_frame.pack(fill=tk.BOTH, expand=True)

        tk.Label(
            options_frame,
            text="الرجاء اختيار نوع المتعاون:",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["surface"]
        ).pack(pady=(0, 20))

        # الخيارات - مع إضافة "متعاون" كخيار افتراضي
        types = [
            ("متعاون", "متعاون"),  # الخيار الافتراضي الجديد
            ("متعاون مدني", "متعاون مدني"),
            ("متعاون عسكري", "متعاون عسكري"),
            ("متعاون عسكري متقاعد", "متعاون عسكري متقاعد")
        ]

        for text, value in types:
            rb = tk.Radiobutton(
                options_frame,
                text=text,
                variable=selected_type,
                value=value,
                font=self.FONTS["text"],
                bg=self.COLORS["surface"],
                activebackground=self.COLORS["surface"],
                selectcolor=self.COLORS["primary"]
            )
            rb.pack(anchor=tk.W, pady=8, padx=20)

        # تحديد الخيار الأول افتراضياً (متعاون)
        selected_type.set("متعاون")

        # النتيجة
        result = {"value": None}

        def confirm():
            result["value"] = selected_type.get()
            dialog.destroy()

        def cancel():
            dialog.destroy()

        # أزرار الإجراءات
        buttons_frame = tk.Frame(dialog, bg=self.COLORS["background"])
        buttons_frame.pack(fill=tk.X, pady=20)

        tk.Button(
            buttons_frame,
            text="موافق",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["success"],
            fg="white",
            padx=30,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            command=confirm
        ).pack(side=tk.LEFT, padx=20)

        tk.Button(
            buttons_frame,
            text="إلغاء",
            font=self.FONTS["text_bold"],
            bg=self.COLORS["danger"],
            fg="white",
            padx=30,
            pady=8,
            bd=0,
            relief=tk.FLAT,
            command=cancel
        ).pack(side=tk.RIGHT, padx=20)

        # انتظار إغلاق النافذة
        dialog.wait_window()

        return result["value"]

    def _import_from_excel(self, category_type):
        """استيراد المدرسين من ملف Excel - محدث بدون الدورات"""
        from tkinter import filedialog
        import pandas as pd
        import openpyxl

        # إذا كان الاختيار متعاون، اسأل عن النوع
        if category_type == "متعاون":
            category = self._select_collaborator_type()
            if not category:
                return
        else:
            category = category_type

        # اختيار ملف Excel
        file_path = filedialog.askopenfilename(
            title=f"اختر ملف Excel لاستيراد {category}",
            filetypes=[("Excel files", "*.xlsx *.xls"), ("All files", "*.*")]
        )

        if not file_path:
            return

        try:
            # قراءة ملف Excel
            df = pd.read_excel(file_path)

            # التحقق من وجود الأعمدة المطلوبة
            required_columns = ["الاسم", "رقم الهوية"]
            optional_columns = ["الرتبة", "رقم الجوال", "جهة العمل", "المؤهل الدراسي"]

            missing_columns = []
            for col in required_columns:
                if col not in df.columns:
                    missing_columns.append(col)

            if missing_columns:
                messagebox.showerror("خطأ", f"الأعمدة المطلوبة غير موجودة: {', '.join(missing_columns)}")
                return

            # إنشاء نافذة التقدم - طولية
            progress_window = tk.Toplevel(self)
            progress_window.title("استيراد المدرسين")
            progress_window.geometry("500x700")
            progress_window.configure(bg=self.COLORS["background"])
            progress_window.transient(self)
            progress_window.grab_set()
            progress_window.resizable(False, True)

            # توسيط النافذة
            progress_window.update_idletasks()
            x = (progress_window.winfo_screenwidth() - 500) // 2
            y = (progress_window.winfo_screenheight() - 700) // 2
            progress_window.geometry(f"500x700+{x}+{y}")

            # عنوان
            header_frame = tk.Frame(progress_window, bg=self.COLORS["primary"], height=60)
            header_frame.pack(fill=tk.X)
            header_frame.pack_propagate(False)

            tk.Label(
                header_frame,
                text=f"استيراد {category}",
                font=self.FONTS["title"],
                bg=self.COLORS["primary"],
                fg="white"
            ).pack(expand=True)

            # إطار المحتوى الرئيسي
            main_frame = tk.Frame(progress_window, bg=self.COLORS["background"])
            main_frame.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

            # إطار التقدم
            progress_frame = tk.Frame(main_frame, bg=self.COLORS["surface"], relief=tk.RIDGE, bd=1)
            progress_frame.pack(fill=tk.X, pady=(0, 10))

            inner_progress = tk.Frame(progress_frame, bg=self.COLORS["surface"], padx=15, pady=15)
            inner_progress.pack(fill=tk.X)

            # شريط التقدم
            from tkinter import ttk
            progress_bar = ttk.Progressbar(
                inner_progress,
                length=450,
                mode='determinate'
            )
            progress_bar.pack()

            # تسمية الحالة
            status_label = tk.Label(
                inner_progress,
                text="جاري الاستيراد...",
                font=self.FONTS["text"],
                bg=self.COLORS["surface"]
            )
            status_label.pack(pady=(5, 0))

            # عدادات
            success_count = 0
            duplicate_list = []
            error_count = 0

            # المؤهلات المعترف بها
            valid_qualifications = ["دكتوراه", "ماجستير", "دبلوم عالي", "بكالوريوس", "جامعة متوسطة", "ثانوي"]

            # معالجة البيانات
            total_rows = len(df)
            progress_bar['maximum'] = total_rows

            cursor = self.db_conn.cursor()

            for index, row in df.iterrows():
                progress_bar['value'] = index + 1
                progress_window.update()

                # الحصول على البيانات
                name = str(row.get("الاسم", "")).strip()
                id_number = str(row.get("رقم الهوية", "")).strip()

                # التحقق من البيانات المطلوبة
                if not name or not id_number:
                    error_count += 1
                    continue

                # التحقق من طول رقم الهوية
                if len(id_number) != 10 or not id_number.isdigit():
                    error_count += 1
                    status_label.config(text=f"خطأ في رقم الهوية: {id_number}")
                    continue

                # التحقق من وجود رقم الهوية مسبقاً
                cursor.execute("SELECT name, rank FROM teachers WHERE id_number = ?", (id_number,))
                existing = cursor.fetchone()

                if existing:
                    duplicate_list.append({
                        'name': existing[0],
                        'rank': existing[1] or "غير محدد",
                        'id': id_number
                    })
                    status_label.config(text=f"مكرر: {name}")
                    continue

                # الحصول على البيانات الاختيارية
                rank = str(row.get("الرتبة", "")).strip()
                phone = str(row.get("رقم الجوال", "")).strip()
                workplace = str(row.get("جهة العمل", "")).strip()

                # التحقق من المؤهل الدراسي
                qualification = str(row.get("المؤهل الدراسي", "")).strip()
                if qualification not in valid_qualifications:
                    qualification = "لا يوجد مؤهل"

                try:
                    # إدراج المدرس مع الفئة الصحيحة
                    cursor.execute("""
                        INSERT INTO teachers (name, rank, id_number, phone, workplace, 
                                            qualification, category, created_date)
                        VALUES (?, ?, ?, ?, ?, ?, ?, ?)
                    """, (name, rank, id_number, phone, workplace, qualification, category,
                          datetime.now().strftime("%Y-%m-%d %H:%M:%S")))

                    success_count += 1
                    status_label.config(text=f"تمت الإضافة: {name}")

                except Exception as e:
                    error_count += 1
                    print(f"خطأ في إضافة {name}: {str(e)}")

            # حفظ التغييرات
            self.db_conn.commit()

            # إخفاء شريط التقدم
            progress_frame.destroy()

            # إطار النتائج
            results_frame = tk.Frame(main_frame, bg=self.COLORS["surface"], relief=tk.RIDGE, bd=1)
            results_frame.pack(fill=tk.BOTH, expand=True)

            # إطار الملخص
            summary_frame = tk.Frame(results_frame, bg="#e8f5e9", padx=15, pady=15)
            summary_frame.pack(fill=tk.X)

            tk.Label(
                summary_frame,
                text="ملخص النتائج",
                font=self.FONTS["subtitle"],
                bg="#e8f5e9"
            ).pack()

            # عرض الإحصائيات
            stats_frame = tk.Frame(summary_frame, bg="#e8f5e9")
            stats_frame.pack(pady=10)

            stats = [
                ("✓ تم إضافة:", success_count, self.COLORS["success"]),
                ("⚠ مكرر:", len(duplicate_list), self.COLORS["warning"]),
                ("✗ أخطاء:", error_count, self.COLORS["danger"])
            ]

            for stat_text, count, color in stats:
                stat_line = tk.Frame(stats_frame, bg="#e8f5e9")
                stat_line.pack(fill=tk.X, pady=2)

                tk.Label(
                    stat_line,
                    text=stat_text,
                    font=self.FONTS["text"],
                    bg="#e8f5e9",
                    width=10,
                    anchor=tk.E
                ).pack(side=tk.LEFT)

                tk.Label(
                    stat_line,
                    text=str(count),
                    font=self.FONTS["text_bold"],
                    bg="#e8f5e9",
                    fg=color,
                    width=5
                ).pack(side=tk.LEFT)

            tk.Label(
                summary_frame,
                text=f"المجموع: {total_rows} سجل",
                font=self.FONTS["text_bold"],
                bg="#e8f5e9"
            ).pack()

            # عرض المكررين إن وجدوا
            if duplicate_list:
                # إطار المكررين
                dup_container = tk.Frame(results_frame, bg=self.COLORS["surface"])
                dup_container.pack(fill=tk.BOTH, expand=True, padx=10, pady=10)

                tk.Label(
                    dup_container,
                    text=f"السجلات المكررة ({len(duplicate_list)})",
                    font=self.FONTS["text_bold"],
                    bg=self.COLORS["surface"],
                    fg=self.COLORS["warning"]
                ).pack(pady=(0, 5))

                # إطار قابل للتمرير
                canvas = tk.Canvas(dup_container, bg=self.COLORS["surface"], height=250)
                scrollbar = tk.Scrollbar(dup_container, orient="vertical", command=canvas.yview)
                scrollable_frame = tk.Frame(canvas, bg=self.COLORS["surface"])

                scrollable_frame.bind(
                    "<Configure>",
                    lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
                )

                canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
                canvas.configure(yscrollcommand=scrollbar.set)

                # عرض المكررين في جدول بسيط
                headers = ["#", "الاسم", "الرتبة", "رقم الهوية"]
                header_frame = tk.Frame(scrollable_frame, bg=self.COLORS["primary"])
                header_frame.pack(fill=tk.X, padx=5, pady=(0, 5))

                for i, header in enumerate(headers):
                    tk.Label(
                        header_frame,
                        text=header,
                        font=self.FONTS["text_bold"],
                        bg=self.COLORS["primary"],
                        fg="white",
                        width=[3, 20, 12, 12][i],
                        anchor=tk.CENTER
                    ).pack(side=tk.LEFT, padx=1)

                # عرض البيانات
                for idx, dup in enumerate(duplicate_list):
                    row_bg = "#fff3cd" if idx % 2 == 0 else "#ffeaa7"
                    row_frame = tk.Frame(scrollable_frame, bg=row_bg)
                    row_frame.pack(fill=tk.X, padx=5, pady=1)

                    # رقم
                    tk.Label(
                        row_frame,
                        text=str(idx + 1),
                        font=self.FONTS["small"],
                        bg=row_bg,
                        width=3,
                        anchor=tk.CENTER
                    ).pack(side=tk.LEFT, padx=1)

                    # الاسم
                    tk.Label(
                        row_frame,
                        text=dup['name'],
                        font=self.FONTS["small"],
                        bg=row_bg,
                        width=20,
                        anchor=tk.W
                    ).pack(side=tk.LEFT, padx=1)

                    # الرتبة
                    tk.Label(
                        row_frame,
                        text=dup['rank'],
                        font=self.FONTS["small"],
                        bg=row_bg,
                        width=12,
                        anchor=tk.CENTER
                    ).pack(side=tk.LEFT, padx=1)

                    # رقم الهوية
                    tk.Label(
                        row_frame,
                        text=dup['id'],
                        font=self.FONTS["small"],
                        bg=row_bg,
                        width=12,
                        anchor=tk.CENTER
                    ).pack(side=tk.LEFT, padx=1)

                canvas.pack(side="left", fill="both", expand=True)
                scrollbar.pack(side="right", fill="y")

            # زر الإغلاق
            close_btn = tk.Button(
                progress_window,
                text="إغلاق",
                font=self.FONTS["text_bold"],
                bg=self.COLORS["primary"],
                fg="white",
                padx=30,
                pady=10,
                bd=0,
                relief=tk.FLAT,
                cursor="hand2",
                command=lambda: [progress_window.destroy(), self._load_teachers()]
            )
            close_btn.pack(pady=10)

        except Exception as e:
            messagebox.showerror("خطأ", f"حدث خطأ أثناء قراءة الملف:\n{str(e)}")
