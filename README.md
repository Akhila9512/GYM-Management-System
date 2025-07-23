# GYM-Management-System
//HTML,CSS,JavaScript.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>GYM Management System</title>
  <style>
    /* Simplified CSS */
    body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background-color: #f5f5f5; }
    .container { max-width: 1000px; margin: 0 auto; }
    .card { background: white; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); padding: 20px; margin-bottom: 20px; }
    .hidden { display: none; }
    table { width: 100%; border-collapse: collapse; margin: 15px 0; }
    th, td { padding: 10px; text-align: left; border-bottom: 1px solid #ddd; }
    th { background-color: #f2f2f2; }
    button, input, select { padding: 8px 12px; margin: 5px 0; }
    button { background: #4285f4; color: white; border: none; border-radius: 4px; cursor: pointer; }
    button:hover { background: #3367d6; }
    .tab { display: flex; border-bottom: 1px solid #ccc; }
    .tab button { background: none; border: none; border-bottom: 3px solid transparent; }
    .tab button.active { border-bottom-color: #4285f4; }
    .tabcontent { display: none; padding: 15px 0; }
    .tabcontent.active { display: block; }
  </style>
</head>
<body>
  <div class="container">
    <!-- Login Screen -->
    <div id="login-container" class="card">
      <h2>GYM Management System</h2>
      <form id="login-form">
        <input type="email" id="email" placeholder="Email" required>
        <input type="password" id="password" placeholder="Password" required>
        <button type="submit">Login</button>
        <div id="login-message"></div>
      </form>
    </div>

    <!-- Admin Dashboard -->
    <div id="admin-dashboard" class="hidden">
      <div class="card">
        <div style="display: flex; justify-content: space-between;">
          <h2>Admin Dashboard</h2>
          <button id="admin-logout">Logout</button>
        </div>
        
        <div class="tab">
          <button class="tablinks active" data-tab="members">Members</button>
          <button class="tablinks" data-tab="bills">Bills</button>
          <button class="tablinks" data-tab="notifications">Notifications</button>
        </div>
        
        <div id="members" class="tabcontent active">
          <button id="add-member-btn">Add Member</button>
          <div id="member-form-container" class="hidden" style="margin-top: 15px;">
            <form id="member-form">
              <input type="text" id="member-name" placeholder="Name" required>
              <input type="email" id="member-email" placeholder="Email" required>
              <button type="submit">Save</button>
              <button type="button" id="cancel-member">Cancel</button>
            </form>
          </div>
          <table id="members-table">
            <thead><tr><th>Name</th><th>Email</th><th>Actions</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
        
        <div id="bills" class="tabcontent">
          <button id="create-bill-btn">Create Bill</button>
          <div id="bill-form-container" class="hidden" style="margin-top: 15px;">
            <form id="bill-form">
              <select id="bill-member" required></select>
              <input type="number" id="bill-amount" placeholder="Amount" required>
              <button type="submit">Create</button>
              <button type="button" id="cancel-bill">Cancel</button>
            </form>
          </div>
          <table id="bills-table">
            <thead><tr><th>Member</th><th>Amount</th><th>Status</th><th>Actions</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
        
        <div id="notifications" class="tabcontent">
          <form id="notification-form">
            <select id="notification-member"><option value="all">All Members</option></select>
            <input type="text" id="notification-title" placeholder="Title" required>
            <textarea id="notification-message" placeholder="Message" required></textarea>
            <button type="submit">Send</button>
          </form>
          <table id="notifications-table" style="margin-top: 15px;">
            <thead><tr><th>Title</th><th>Message</th><th>Date</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
      </div>
    </div>

    <!-- Member Dashboard -->
    <div id="member-dashboard" class="hidden">
      <div class="card">
        <div style="display: flex; justify-content: space-between;">
          <h2>Member Dashboard</h2>
          <button id="member-logout">Logout</button>
        </div>
        
        <div class="tab">
          <button class="tablinks active" data-tab="profile">Profile</button>
          <button class="tablinks" data-tab="my-bills">My Bills</button>
        </div>
        
        <div id="profile" class="tabcontent active">
          <div id="member-profile"></div>
        </div>
        
        <div id="my-bills" class="tabcontent">
          <table id="member-bills-table">
            <thead><tr><th>Amount</th><th>Due Date</th><th>Status</th></tr></thead>
            <tbody></tbody>
          </table>
        </div>
      </div>
    </div>
  </div>

  <!-- Firebase SDK (load only needed modules) -->
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-auth.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-firestore.js"></script>

  <script>
    // Optimized Firebase implementation
    document.addEventListener('DOMContentLoaded', function() {
      // 1. INITIALIZE FIREBASE (REPLACE WITH YOUR CONFIG)
      const firebaseConfig = {
        apiKey: "YOUR_API_KEY",
        authDomain: "YOUR_AUTH_DOMAIN",
        projectId: "YOUR_PROJECT_ID",
        storageBucket: "YOUR_STORAGE_BUCKET",
        messagingSenderId: "YOUR_SENDER_ID",
        appId: "YOUR_APP_ID"
      };
      
      // Initialize only what we need
      firebase.initializeApp(firebaseConfig);
      const auth = firebase.auth();
      const db = firebase.firestore();
      
      // Enable offline persistence for better performance
      db.enablePersistence().catch(err => {
        console.log('Firebase offline persistence error:', err);
      });
      
      // 2. CACHE DOM ELEMENTS
      const elements = {
        login: document.getElementById('login-container'),
        admin: document.getElementById('admin-dashboard'),
        member: document.getElementById('member-dashboard'),
        loginForm: document.getElementById('login-form'),
        loginMsg: document.getElementById('login-message'),
        // ... add other elements as needed
      };
      
      // 3. AUTHENTICATION
      let currentUser = null;
      
      // Auth state listener
      auth.onAuthStateChanged(async user => {
        if (user) {
          // Get user role
          const userDoc = await db.collection('users').doc(user.uid).get();
          if (userDoc.exists) {
            currentUser = {
              uid: user.uid,
              email: user.email,
              role: userDoc.data().role
            };
            
            // Show appropriate dashboard
            elements.login.classList.add('hidden');
            elements.admin.classList.add('hidden');
            elements.member.classList.add('hidden');
            
            if (currentUser.role === 'admin') {
              elements.admin.classList.remove('hidden');
              initAdminDashboard();
            } else if (currentUser.role === 'member') {
              elements.member.classList.remove('hidden');
              initMemberDashboard();
            }
          } else {
            await auth.signOut();
            showMessage('User record not found', 'error');
          }
        } else {
          // Not logged in
          currentUser = null;
          elements.login.classList.remove('hidden');
          elements.admin.classList.add('hidden');
          elements.member.classList.add('hidden');
        }
      });
      
      // Login form
      elements.loginForm.addEventListener('submit', async e => {
        e.preventDefault();
        const email = document.getElementById('email').value;
        const password = document.getElementById('password').value;
        
        try {
          await auth.signInWithEmailAndPassword(email, password);
          showMessage('Login successful', 'success');
        } catch (error) {
          showMessage(error.message, 'error');
        }
      });
      
      // 4. ADMIN DASHBOARD FUNCTIONS
      function initAdminDashboard() {
        // Tab switching
        document.querySelectorAll('.tablinks').forEach(btn => {
          btn.addEventListener('click', function() {
            const tabId = this.getAttribute('data-tab');
            document.querySelectorAll('.tabcontent').forEach(tab => {
              tab.classList.remove('active');
            });
            document.querySelectorAll('.tablinks').forEach(tab => {
              tab.classList.remove('active');
            });
            document.getElementById(tabId).classList.add('active');
            this.classList.add('active');
          });
        });
        
        // Load members
        loadMembers();
        
        // Member form toggle
        document.getElementById('add-member-btn').addEventListener('click', () => {
          document.getElementById('member-form-container').classList.remove('hidden');
        });
        
        document.getElementById('cancel-member').addEventListener('click', () => {
          document.getElementById('member-form-container').classList.add('hidden');
          document.getElementById('member-form').reset();
        });
        
        // Member form submission
        document.getElementById('member-form').addEventListener('submit', async e => {
          e.preventDefault();
          const name = document.getElementById('member-name').value;
          const email = document.getElementById('member-email').value;
          
          try {
            await db.collection('members').add({
              name,
              email,
              createdAt: firebase.firestore.FieldValue.serverTimestamp()
            });
            
            document.getElementById('member-form').reset();
            document.getElementById('member-form-container').classList.add('hidden');
            loadMembers();
            showMessage('Member added successfully', 'success');
          } catch (error) {
            showMessage('Error adding member: ' + error.message, 'error');
          }
        });
        
        // Logout
        document.getElementById('admin-logout').addEventListener('click', () => auth.signOut());
      }
      
      // Optimized member loading with pagination
      async function loadMembers() {
        const tbody = document.querySelector('#members-table tbody');
        tbody.innerHTML = '<tr><td colspan="3">Loading...</td></tr>';
        
        try {
          const snapshot = await db.collection('members')
            .orderBy('createdAt', 'desc')
            .limit(20) // Limit to 20 for performance
            .get();
          
          if (snapshot.empty) {
            tbody.innerHTML = '<tr><td colspan="3">No members found</td></tr>';
            return;
          }
          
          tbody.innerHTML = '';
          snapshot.forEach(doc => {
            const member = doc.data();
            const row = document.createElement('tr');
            row.innerHTML = `
              <td>${member.name}</td>
              <td>${member.email}</td>
              <td>
                <button data-id="${doc.id}" class="edit-member">Edit</button>
                <button data-id="${doc.id}" class="delete-member">Delete</button>
              </td>
            `;
            tbody.appendChild(row);
          });
          
          // Add event listeners
          document.querySelectorAll('.edit-member').forEach(btn => {
            btn.addEventListener('click', () => editMember(btn.getAttribute('data-id')));
          });
          
          document.querySelectorAll('.delete-member').forEach(btn => {
            btn.addEventListener('click', () => deleteMember(btn.getAttribute('data-id')));
          });
          
          // Populate member dropdowns
          populateMemberDropdowns(snapshot);
        } catch (error) {
          showMessage('Error loading members: ' + error.message, 'error');
        }
      }
      
      // 5. MEMBER DASHBOARD FUNCTIONS
      function initMemberDashboard() {
        // Tab switching (same as admin)
        document.querySelectorAll('.tablinks').forEach(btn => {
          btn.addEventListener('click', function() {
            const tabId = this.getAttribute('data-tab');
            document.querySelectorAll('.tabcontent').forEach(tab => {
              tab.classList.remove('active');
            });
            document.querySelectorAll('.tablinks').forEach(tab => {
              tab.classList.remove('active');
            });
            document.getElementById(tabId).classList.add('active');
            this.classList.add('active');
          });
        });
        
        // Load member data
        loadMemberProfile();
        loadMemberBills();
        
        // Logout
        document.getElementById('member-logout').addEventListener('click', () => auth.signOut());
      }
      
      // 6. UTILITY FUNCTIONS
      function showMessage(message, type = 'info') {
        elements.loginMsg.textContent = message;
        elements.loginMsg.className = type;
        setTimeout(() => {
          elements.loginMsg.textContent = '';
          elements.loginMsg.className = '';
        }, 3000);
      }
      
      function populateMemberDropdowns(snapshot) {
        const billMemberSelect = document.getElementById('bill-member');
        const notificationMemberSelect = document.getElementById('notification-member');
        
        // Clear existing options (keep first option if it exists)
        billMemberSelect.innerHTML = '<option value="">Select Member</option>';
        notificationMemberSelect.innerHTML = '<option value="all">All Members</option>';
        
        snapshot.forEach(doc => {
          const member = doc.data();
          
          const option1 = document.createElement('option');
          option1.value = doc.id;
          option1.textContent = member.name;
          billMemberSelect.appendChild(option1);
          
          const option2 = document.createElement('option');
          option2.value = doc.id;
          option2.textContent = member.name;
          notificationMemberSelect.appendChild(option2);
        });
      }
    });
  </script>
</body>
</html>
