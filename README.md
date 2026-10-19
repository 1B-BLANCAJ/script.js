# script.js

const pages = [...document.querySelectorAll(".page")];
const navButtons = [
    ...document.querySelectorAll("nav button[data-page]")
];

const dashNav = document.getElementById("dashNav");
const attendanceForm = document.getElementById("attendanceForm");
const loginForm = document.getElementById("loginForm");

const attMsg = document.getElementById("attMsg");
const loginMsg = document.getElementById("loginMsg");

const sid = document.getElementById("sid");
const sname = document.getElementById("sname");
const section = document.getElementById("section");
const lab = document.getElementById("lab");
const code = document.getElementById("code");

const username = document.getElementById("username");
const password = document.getElementById("password");

const sessStatus = document.getElementById("sessStatus");
const sessCode = document.getElementById("sessCode");
const timer = document.getElementById("timer");
const sessNote = document.getElementById("sessNote");

const total = document.getElementById("total");
const present = document.getElementById("present");
const late = document.getElementById("late");
const rows = document.getElementById("rows");

let logged = false;
let activeCode = "";
let expires = 0;
let timerId = null;
let records = [];

// PAGE NAVIGATION
function show(id) {
    pages.forEach(page => {
        page.classList.toggle("active", page.id === id);
    });

    navButtons.forEach(button => {
        button.classList.toggle(
            "active",
            button.dataset.page === id
        );
    });
}

navButtons.forEach(button => {
    button.addEventListener("click", () => {
        if (button.dataset.page === "dashboard" && !logged) {
            show("login");
            return;
        }

        show(button.dataset.page);
    });
});

// DISPLAY MESSAGES
function msg(element, kind, text) {
    element.className = "message " + kind;
    element.textContent = text;
}

// INSTRUCTOR LOGIN
loginForm.addEventListener("submit", event => {
    event.preventDefault();

    if (
        username.value.trim() === "instructor" &&
        password.value === "1234"
    ) {
        logged = true;
        dashNav.hidden = false;
        loginMsg.className = "message";

        show("dashboard");
    } else {
        msg(
            loginMsg,
            "error",
            "Incorrect username or password. Please try again."
        );
    }
});

// UPDATE ATTENDANCE TIMER
function tick() {
    const left = Math.max(
        0,
        Math.ceil((expires - Date.now()) / 1000)
    );

    timer.textContent = left;

    if (left === 0) {
        activeCode = "";
        sessStatus.textContent = "Closed";
        sessCode.textContent = "------";

        clearInterval(timerId);
        timerId = null;
    }
}

// START ATTENDANCE SESSION
document.getElementById("start").addEventListener("click", () => {
    activeCode = String(
        Math.floor(100000 + Math.random() * 900000)
    );

    expires = Date.now() + 300000;

    sessStatus.textContent = "Open";
    sessCode.textContent = activeCode;

    sessNote.textContent =
        "Show this code in the lab. It expires in 5 minutes (prototype demo).";

    if (timerId) {
        clearInterval(timerId);
    }

    tick();
    timerId = setInterval(tick, 250);
});

// SUBMIT STUDENT ATTENDANCE
attendanceForm.addEventListener("submit", event => {
    event.preventDefault();

    if (!activeCode || Date.now() >= expires) {
        msg(
            attMsg,
            "error",
            "No active session. Ask the instructor to start attendance first."
        );
        return;
    }

    if (code.value.trim() !== activeCode) {
        msg(
            attMsg,
            "error",
            "Incorrect attendance code. Check the code on the lab screen."
        );
        return;
    }

    const studentId = sid.value.trim();

    if (
        records.some(
            record => record.id.toLowerCase() ===
                studentId.toLowerCase()
        )
    ) {
        msg(
            attMsg,
            "error",
            "This Student ID has already been recorded in this demo."
        );
        return;
    }

    const now = new Date();

    records.push({
        id: studentId,
        name: sname.value.trim(),
        section: section.value,
        lab: lab.value,
        time: now.toLocaleTimeString(),
        status: "Present"
    });

    render();

    msg(
        attMsg,
        "success",
        "Attendance confirmed at " + now.toLocaleTimeString() + "."
    );

    attendanceForm.reset();
});

// PREVENT HTML INJECTION IN TABLE VALUES
function esc(value) {
    return String(value).replace(/[&<>"']/g, character => ({
        "&": "&amp;",
        "<": "&lt;",
        ">": "&gt;",
        '"': "&quot;",
        "'": "&#39;"
    })[character]);
}

// DISPLAY ATTENDANCE RECORDS
function render() {
    total.textContent = records.length;

    present.textContent = records.filter(
        record => record.status === "Present"
    ).length;

    late.textContent = records.filter(
        record => record.status === "Late"
    ).length;

    if (records.length === 0) {
        rows.innerHTML = `
            <tr>
                <td colspan="6">No attendance recorded yet.</td>
            </tr>
        `;
        return;
    }

    rows.innerHTML = records.map(record => `
        <tr>
            <td>${esc(record.id)}</td>
            <td>${esc(record.name)}</td>
            <td>${esc(record.section)}</td>
            <td>${esc(record.lab)}</td>
            <td>${esc(record.time)}</td>
            <td>${esc(record.status)}</td>
        </tr>
    `).join("");
}

// CLEAR DEMO RECORDS
document.getElementById("clear").addEventListener("click", () => {
    if (confirm("Clear all demo attendance records?")) {
        records = [];
        render();
    }
});

// LOGOUT
document.getElementById("logout").addEventListener("click", () => {
    logged = false;
    dashNav.hidden = true;

    loginForm.reset();
    loginMsg.className = "message";

    show("student");
});
