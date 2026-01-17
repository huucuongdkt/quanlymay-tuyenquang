<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Hệ Thống Quản Lý V9.3 Turbo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://unpkg.com/html5-qrcode" type="text/javascript"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" />
    <style>
        :root { --main-color: #1a237e; }
        body { background-color: #f3f4f6; font-family: 'Segoe UI', sans-serif; -webkit-tap-highlight-color: transparent; }
        
        #login-screen { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: #1a237e; z-index: 9999; display: flex; justify-content: center; align-items: center; flex-direction: column; }
        .login-box { background: white; padding: 25px; border-radius: 15px; width: 85%; max-width: 350px; text-align: center; box-shadow: 0 10px 25px rgba(0,0,0,0.3); }
        
        .bottom-nav { position: fixed; bottom: 0; left: 0; width: 100%; background: white; border-top: 1px solid #eee; display: none; justify-content: space-around; padding: 10px 0; z-index: 100; box-shadow: 0 -2px 10px rgba(0,0,0,0.05); }
        .nav-item { text-align: center; color: #999; font-size: 12px; cursor: pointer; flex: 1; }
        .nav-item.active { color: var(--main-color); font-weight: bold; }
        .nav-item i { font-size: 20px; display: block; margin-bottom: 3px; }

        .screen { display: none; padding-bottom: 20px; }
        .has-nav .screen { padding-bottom: 80px; }
        .screen.active { display: block; }
        
        .form-section { background: white; padding: 12px; border-radius: 12px; margin-bottom: 15px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .form-input { width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 6px; font-size: 14px; }
        .btn-submit { background-color: var(--main-color); color: white; width: 100%; padding: 12px; border-radius: 8px; font-weight: bold; margin-top: 5px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); }
        .form-label-group { display: flex; justify-content: space-between; align-items: center; margin-bottom: 5px; }
        .form-label { font-weight: 600; color: #333; font-size: 13px; }
        .btn-refresh { color: var(--main-color); cursor: pointer; padding: 4px; transition: 0.3s; }
        .btn-refresh:active { transform: rotate(180deg); }

        .stat-card { background: white; padding: 15px; border-radius: 12px; margin-bottom: 10px; border-left: 5px solid var(--main-color); box-shadow: 0 2px 4px rgba(0,0,0,0.05); }
        .stat-value { font-size: 24px; font-weight: bold; color: var(--main-color); }
        .history-item { border-bottom: 1px solid #eee; padding: 10px 0; font-size: 13px; }
        
        /* Loading Overlay để chặn thao tác khi đang gửi */
        #loading-overlay { display: none; position: fixed; top:0; left:0; width:100%; height:100%; background: rgba(255,255,255,0.7); z-index: 10000; justify-content: center; align-items: center; }
    </style>
</head>
<body>

    <div id="loading-overlay"><div class="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-900"></div></div>

    <div id="login-screen">
        <div class="login-box">
            <h2 class="text-2xl font-bold text-blue-900 mb-4 uppercase">Đăng Nhập</h2>
            <input type="text" id="username" class="form-input mb-3" placeholder="Tài khoản (SĐT)">
            <input type="password" id="password" class="form-input mb-4" placeholder="Mật khẩu">
            <button onclick="doLogin()" class="btn-submit w-full">VÀO HỆ THỐNG</button>
        </div>
        <p class="text-white text-xs mt-4 opacity-70">Turbo Speed V9.3</p>
    </div>

    <div class="bg-[#1a237e] text-white p-4 sticky top-0 z-50 shadow flex justify-between items-center">
        <div class="font-bold uppercase text-lg" id="header-title">NHẬT TRÌNH</div>
        <div class="text-xs bg-blue-800 px-3 py-1 rounded cursor-pointer hover:bg-red-600 transition" onclick="doLogout()">
            <i class="fas fa-sign-out-alt mr-1"></i> Thoát
        </div>
    </div>

    <div id="screen-input" class="screen active container mx-auto p-3 max-w-lg">
        <form id="reportForm" onsubmit="submitForm(event)">
            
            <div class="form-section">
                <div class="bg-blue-50 p-2 rounded mb-3 text-sm text-blue-900 font-semibold border border-blue-100">
                    <i class="fas fa-id-badge mr-1"></i> <span id="user-fullname">...</span>
                </div>
                <div class="grid grid-cols-2 gap-3 mb-3">
                    <div>
                        <label class="form-label block mb-1">Ngày báo cáo</label>
                        <input type="date" id="ngayBaoCao" class="form-input" required>
                    </div>
                    <div>
                        <label class="form-label block mb-1">SĐT liên hệ</label>
                        <input type="tel" id="sdt" class="form-input" placeholder="09xxxx..." required>
                    </div>
                </div>

                <div class="mb-3">
                    <div class="form-label-group">
                        <label class="form-label">Tên Công Trình</label>
                        <i class="fas fa-sync-alt btn-refresh" onclick="refreshData(this, true)" title="Cập nhật"></i>
                    </div>
                    <select id="tenCongTrinh" class="form-input font-semibold text-blue-800" required><option value="">-- Đang tải... --</option></select>
                </div>

                <div class="mb-1">
                    <div class="form-label-group">
                        <label class="form-label">Thiết bị / Xe</label>
                        <i class="fas fa-sync-alt btn-refresh" onclick="refreshData(this, true)" title="Cập nhật"></i>
                    </div>
                    <div class="flex gap-2">
                        <select id="maThietBi" class="form-input flex-1" required><option value="">-- Đang tải... --</option></select>
                        <button type="button" onclick="startScan()" class="bg-[#1a237e] text-white px-3 rounded-lg"><i class="fas fa-qrcode"></i></button>
                    </div>
                </div>
            </div>

            <div class="form-section">
                <div class="form-label-group border-b pb-1 mb-2">
                    <span class="font-bold text-blue-900 text-sm">Nội dung công việc</span>
                    <button type="button" onclick="addWorkRow()" class="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded font-bold">+ Thêm</button>
                </div>
                <div id="workTableBody"></div>
            </div>
            
            <div class="form-section">
                <div class="form-label-group border-b pb-1 mb-2">
                    <span class="font-bold text-blue-900 text-sm">Nhiên liệu & Vật tư</span>
                    <button type="button" onclick="addFuelRow()" class="text-xs bg-blue-100 text-blue-800 px-2 py-1 rounded font-bold">+ Thêm</button>
                </div>
                <div class="flex gap-1 mb-1 text-xs text-gray-500 font-semibold px-1">
                    <div class="w-5/12">Loại</div>
                    <div class="w-3/12 text-center">SL</div>
                    <div class="w-3/12">Đơn vị</div>
                    <div class="w-1/12"></div>
                </div>
                <div id="fuelTableBody"></div>
            </div>

            <div class="form-section border-l-4 border-red-500">
                <label class="block text-red-700 font-bold mb-1 text-sm">Sửa chữa & Kiến nghị</label>
                <textarea id="noiDungSua" class="form-input mb-2" rows="1" placeholder="Hư hỏng..."></textarea>
                <input type="number" id="chiPhiSua" class="form-input mb-2" placeholder="Chi phí (VNĐ)">
                <textarea id="kienNghi" class="form-input" rows="1" placeholder="Đề xuất..."></textarea>
            </div>

            <button type="submit" class="btn-submit"><i class="fas fa-paper-plane mr-2"></i> GỬI BÁO CÁO</button>
        </form>
    </div>

    <div id="screen-stats" class="screen container mx-auto p-3 max-w-lg">
        <div class="flex justify-between items-center mb-3">
            <h3 class="font-bold text-gray-700 uppercase">TỔNG HỢP</h3>
            <button onclick="loadStats()" class="text-xs bg-white border border-blue-900 text-blue-900 px-2 py-1 rounded"><i class="fas fa-sync"></i> Cập nhật</button>
        </div>
        <div class="grid grid-cols-2 gap-3 mb-4">
            <div class="stat-card">
                <div class="text-sm text-gray-500">Tổng Giờ</div>
                <div class="stat-value" id="stat-hours">0</div>
            </div>
            <div class="stat-card border-l-green-500">
                <div class="text-sm text-gray-500">Tổng Ca</div>
                <div class="stat-value text-green-600" id="stat-shifts">0</div>
            </div>
        </div>
        <div class="bg-white rounded-xl shadow p-4 mb-4">
            <h4 class="font-bold text-red-700 border-b pb-2 mb-2">Sửa chữa gần đây</h4>
            <div id="list-repairs" class="text-sm text-gray-600">Đang tải...</div>
        </div>
        <div class="bg-white rounded-xl shadow p-4">
            <h4 class="font-bold text-yellow-600 border-b pb-2 mb-2">Kiến nghị</h4>
            <div id="list-petitions" class="text-sm text-gray-600">Đang tải...</div>
        </div>
    </div>

    <div class="bottom-nav" id="bottom-nav">
        <div class="nav-item active" id="nav-item-input" onclick="switchScreen('input')">
            <i class="fas fa-edit"></i> Nhập liệu
        </div>
        <div class="nav-item" id="nav-item-stats" onclick="switchScreen('stats')">
            <i class="fas fa-chart-pie"></i> Tổng quát
        </div>
    </div>
    
    <div id="qr-modal" style="display:none;position:fixed;top:0;left:0;width:100%;height:100%;background:rgba(0,0,0,0.9);z-index:9999;justify-content:center;align-items:center;flex-direction:column;">
        <div id="qr-reader" style="width:300px;background:white;padding:10px;border-radius:8px;"></div>
        <button onclick="stopScan()" style="margin-top:20px;color:white;border:1px solid white;padding:8px 20px;border-radius:20px;">Đóng Camera</button>
    </div>

    <script>
        // 👉 LINK API (Đã có Cache)
        const API_URL = "https://script.google.com/macros/s/AKfycbyl5PcdJUh7GjoX0m-SzADXDyX3wOXVRcrz6gkA2H_uCZ9XjnOPQ1lSf1eXd3JUxhGo/exec";
        
        let currentUser = null;
        let html5QrcodeScanner = null;
        const Toast = Swal.mixin({ toast: true, position: 'top-end', showConfirmButton: false, timer: 2000 });

        // TỰ ĐỘNG ĐĂNG NHẬP KHI MỞ APP
        document.addEventListener("DOMContentLoaded", () => {
            const savedUser = localStorage.getItem("user_session");
            if (savedUser) {
                try {
                    const userData = JSON.parse(savedUser);
                    // Giả lập đăng nhập thành công từ bộ nhớ
                    loginSuccess(userData);
                } catch(e) { localStorage.removeItem("user_session"); }
            }
        });

        // --- 1. XỬ LÝ ĐĂNG NHẬP ---
        function doLogin() {
            const u = document.getElementById('username').value;
            const p = document.getElementById('password').value;
            if(!u || !p) return Swal.fire('Lỗi', 'Nhập thiếu thông tin', 'warning');

            Swal.fire({title: 'Đang đăng nhập...', didOpen: () => Swal.showLoading()});

            fetch(API_URL + `?action=login&user=${u}&pass=${p}`)
            .then(r => r.json())
            .then(data => {
                Swal.close();
                if(data.status == 'success') {
                    // Lưu phiên đăng nhập vào điện thoại
                    localStorage.setItem("user_session", JSON.stringify(data));
                    loginSuccess(data);
                } else {
                    Swal.fire('Lỗi', data.message, 'error');
                }
            }).catch(e => Swal.fire('Lỗi', 'Không kết nối được Server', 'error'));
        }

        function loginSuccess(data) {
            currentUser = data;
            document.getElementById('login-screen').style.display = 'none';
            document.getElementById('user-fullname').innerText = data.hoTen;
            
            // Xử lý quyền
            const nav = document.getElementById('bottom-nav');
            const body = document.body;
            const headerTitle = document.getElementById('header-title');

            if (data.quyen === 'Giam_Doc' || data.quyen === 'Quan_Ly') {
                nav.style.display = 'flex';
                body.classList.add('has-nav');
                headerTitle.innerText = "QUẢN TRỊ VIÊN";
                switchScreen('stats');
            } else {
                nav.style.display = 'none';
                body.classList.remove('has-nav');
                switchScreen('input');
                if(data.quyen === 'Lai_Xe') headerTitle.innerText = "NHẬT TRÌNH LÁI XE";
                else if(data.quyen === 'Tram_Tron') headerTitle.innerText = "NHẬT TRÌNH TRẠM TRỘN";
                else headerTitle.innerText = "NHẬT TRÌNH MÁY";
            }

            // Tải dữ liệu từ LocalStorage trước cho nhanh
            loadCachedData();
            document.getElementById('ngayBaoCao').valueAsDate = new Date();
            addWorkRow(); addFuelRow();
        }

        function doLogout() {
            localStorage.removeItem("user_session");
            location.reload();
        }

        // --- 2. HÀM TẢI DỮ LIỆU THÔNG MINH (CACHE FRONTEND) ---
        function loadCachedData() {
            // Lấy từ bộ nhớ điện thoại ra dùng ngay
            const cachedProjects = localStorage.getItem("projects_list");
            const cachedMachines = localStorage.getItem("machines_list");
            
            if(cachedProjects) fillSelect('tenCongTrinh', JSON.parse(cachedProjects));
            if(cachedMachines) fillMachineSelect(JSON.parse(cachedMachines));

            // Âm thầm cập nhật dữ liệu mới từ Server
            refreshData(null, false); 
        }

        async function refreshData(btn, showToast = true) {
            if(btn) btn.classList.add('fa-spin');
            
            try {
                // Gọi song song
                const [pRes, mRes] = await Promise.all([
                    fetch(API_URL + "?action=getDanhSachCongTrinh"),
                    fetch(API_URL + "?action=getDanhSachMay")
                ]);
                
                const projects = await pRes.json();
                const machines = await mRes.json();

                // Lưu vào bộ nhớ điện thoại
                if(Array.isArray(projects)) {
                    localStorage.setItem("projects_list", JSON.stringify(projects));
                    fillSelect('tenCongTrinh', projects);
                }
                if(Array.isArray(machines)) {
                    localStorage.setItem("machines_list", JSON.stringify(machines));
                    fillMachineSelect(machines);
                }
                
                if(btn) btn.classList.remove('fa-spin');
                if(showToast) Toast.fire({icon: 'success', title: 'Đã cập nhật dữ liệu'});

            } catch(e) {
                console.error(e);
                if(btn) btn.classList.remove('fa-spin');
            }
        }

        function fillSelect(id, data) {
            const sel = document.getElementById(id);
            sel.innerHTML = '<option value="">-- Chọn --</option>';
            data.forEach(item => sel.innerHTML += `<option value="${item}">${item}</option>`);
        }
        function fillMachineSelect(data) {
            const sel = document.getElementById('maThietBi');
            sel.innerHTML = '<option value="">-- Chọn --</option>';
            data.forEach(m => sel.innerHTML += `<option value="${m.maMay}">${m.tenMay} (${m.bienSo})</option>`);
        }

        // --- 3. GỬI BÁO CÁO ---
        function submitForm(e) {
            e.preventDefault();
            // Show loading
            document.getElementById('loading-overlay').style.display = 'flex';
            
            let works = [];
            document.querySelectorAll('.work-row').forEach(row => {
                let c = row.querySelector('.w-content').value, v = row.querySelector('.w-vol').value;
                if(c || v) works.push({caLam:row.querySelector('.w-shift').value, noiDung:c||"KĐ", khoiLuong:v||0, donVi:row.querySelector('.w-unit').value});
            });

            let fuel = {diesel:0, dauCau:0, dauMay:0, thuyLuc:0, khac:[]};
            document.querySelectorAll('.fuel-row').forEach(row => {
                let t = row.querySelector('.f-type').value;
                let v = parseFloat(row.querySelector('.f-vol').value) || 0;
                let u = row.querySelector('.f-unit').value;
                if(v>0) {
                    if(t=='Diesel') fuel.diesel+=v; else if(t=='Dầu cầu') fuel.dauCau+=v;
                    else if(t=='Dầu máy') fuel.dauMay+=v; else if(t=='Thủy lực') fuel.thuyLuc+=v;
                    else fuel.khac.push(`${t}: ${v} ${u}`);
                }
            });

            const data = {
                ngayBaoCao: document.getElementById('ngayBaoCao').value,
                tenCongTrinh: document.getElementById('tenCongTrinh').value,
                sdt: document.getElementById('sdt').value || currentUser.u, // Lấy SĐT từ input hoặc tài khoản
                maThietBi: document.getElementById('maThietBi').value,
                nguoiBao: currentUser.hoTen,
                danhSachViec: works, tongKhoiLuong: "Xem chi tiết",
                diesel: fuel.diesel, dauCau: fuel.dauCau, dauThuyLuc: fuel.thuyLuc, dauMay: fuel.dauMay, dauKhac: fuel.khac.join(', '),
                noiDungSua: document.getElementById('noiDungSua').value,
                chiPhiSua: document.getElementById('chiPhiSua').value,
                kienNghi: document.getElementById('kienNghi').value
            };

            fetch(API_URL, {method:'POST', mode:'no-cors', headers:{'Content-Type':'text/plain'}, body:JSON.stringify(data)})
            .then(() => {
                document.getElementById('loading-overlay').style.display = 'none';
                Swal.fire({icon:'success', title:'Thành công', timer:1500, showConfirmButton:false});
                document.getElementById('reportForm').reset();
                document.getElementById('workTableBody').innerHTML=''; addWorkRow();
                document.getElementById('fuelTableBody').innerHTML=''; addFuelRow();
                document.getElementById('ngayBaoCao').valueAsDate=new Date();
            }).catch(() => {
                document.getElementById('loading-overlay').style.display = 'none';
                Swal.fire('Lỗi', 'Gửi thất bại', 'error');
            });
        }

        // --- CÁC HÀM UI KHÁC ---
        function addWorkRow() {
            document.getElementById('workTableBody').insertAdjacentHTML('beforeend', `
            <div class="flex gap-1 mb-2 items-center work-row">
                <select class="form-input w-3/12 w-shift font-bold p-1 text-xs"><option>Sáng</option><option>Chiều</option><option>Tăng ca</option></select>
                <input class="form-input w-4/12 w-content p-1" placeholder="Việc...">
                <input type="number" class="form-input w-2/12 w-vol p-1 text-center" placeholder="SL">
                <select class="form-input w-2/12 w-unit p-1 text-xs"><option>Giờ</option><option>Ca</option><option>m3</option><option>Chuyến</option><option>Km</option></select>
                <div class="w-1/12 text-center"><i class="fas fa-times text-red-500 cursor-pointer" onclick="this.parentElement.parentElement.remove()"></i></div>
            </div>`);
        }
        function addFuelRow() {
            document.getElementById('fuelTableBody').insertAdjacentHTML('beforeend', `
            <div class="flex gap-1 mb-2 items-center fuel-row">
                <select class="form-input w-5/12 f-type font-semibold p-1">
                    <option>Diesel</option><option>Dầu cầu</option><option>Dầu máy</option><option>Thủy lực</option><option>Mỡ</option><option>Nước làm mát</option><option>Khác</option>
                </select>
                <input type="number" class="form-input w-3/12 f-vol p-1 text-center" placeholder="0">
                <select class="form-input w-3/12 f-unit p-1 text-xs"><option value="Lít">Lít</option><option value="Kg">Kg</option><option value="Can">Can</option><option value="Khác">Khác</option></select>
                <div class="w-1/12 text-center"><i class="fas fa-times text-red-500 cursor-pointer" onclick="this.parentElement.parentElement.remove()"></i></div>
            </div>`);
        }
        function switchScreen(name) {
            document.querySelectorAll('.screen').forEach(el => el.classList.remove('active'));
            document.getElementById('screen-' + name).classList.add('active');
            document.querySelectorAll('.nav-item').forEach(el => el.classList.remove('active'));
            const nav = document.getElementById('nav-item-' + name);
            if(nav) nav.classList.add('active');
            if(name === 'stats') loadStats();
        }
        function loadStats() {
            if(!currentUser) return;
            document.getElementById('list-repairs').innerText = 'Đang tải...';
            fetch(API_URL + `?action=getStats&name=${encodeURIComponent(currentUser.hoTen)}&role=${currentUser.quyen}`)
            .then(r => r.json())
            .then(data => {
                document.getElementById('stat-hours').innerText = data.tongGio;
                document.getElementById('stat-shifts').innerText = data.tongCa;
                const d=document.getElementById('list-repairs');
                if(data.suaChua.length==0)d.innerText="Chưa có dữ liệu";
                else d.innerHTML=data.suaChua.map(i=>`<div class="history-item"><div class="text-xs text-gray-500">${i.ngay.substring(0,10)}</div><div>${i.noiDung}</div></div>`).join('');
                
                const p=document.getElementById('list-petitions');
                if(data.kienNghi.length==0)p.innerText="Chưa có dữ liệu";
                else p.innerHTML=data.kienNghi.map(i=>`<div class="history-item"><div class="text-xs text-gray-500">${i.ngay.substring(0,10)}</div><div>${i.noiDung}</div></div>`).join('');
            });
        }
        function startScan() { document.getElementById('qr-modal').style.display='flex'; if(html5QrcodeScanner===null) html5QrcodeScanner=new Html5Qrcode("qr-reader"); html5QrcodeScanner.start({facingMode:"environment"},{fps:10,qrbox:{width:250,height:250}},(t)=>{stopScan(); let s=document.getElementById('maThietBi'); for(let i=0;i<s.options.length;i++) if(s.options[i].value===t) {s.selectedIndex=i; Swal.fire('Đã chọn',t,'success'); return;} Swal.fire('Lỗi','Không tìm thấy máy','warning'); }).catch(()=>{Swal.fire('Lỗi','Cần quyền camera','error');stopScan();}); }
        function stopScan() { if(html5QrcodeScanner) html5QrcodeScanner.stop().then(()=>document.getElementById('qr-modal').style.display='none').catch(()=>document.getElementById('qr-modal').style.display='none'); else document.getElementById('qr-modal').style.display='none'; }
    </script>
</body>
</html>
