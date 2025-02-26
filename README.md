<!DOCTYPE html>
<html lang="vi">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Máy Tính Giải Tích</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        background-color: #eef2f5;
        padding: 20px;
        text-align: center;
      }
      .container {
        max-width: 600px;
        margin: auto;
        background: #fff;
        border-radius: 8px;
        padding: 20px;
        box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
      }
      .expression-input {
        width: 90%;
        padding: 10px;
        font-size: 22px;
        text-align: right;
        margin-bottom: 15px;
        border: 1px solid #ccc;
        border-radius: 4px;
      }
      .btn {
        padding: 10px 20px;
        font-size: 18px;
        background-color: #4caf50;
        color: #fff;
        border: none;
        border-radius: 4px;
        cursor: pointer;
        margin-bottom: 20px;
      }
      .result {
        font-size: 24px;
        font-weight: bold;
        margin-bottom: 15px;
      }
      .explanation {
        text-align: left;
        background: #f9f9f9;
        padding: 15px;
        border: 1px solid #ddd;
        border-radius: 4px;
        font-size: 18px;
        line-height: 1.6;
      }
      /* Style cho bàn phím ảo */
      .keyboard {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 10px;
        margin-bottom: 20px;
      }
      .key {
        padding: 15px;
        font-size: 18px;
        border: 1px solid #ccc;
        border-radius: 4px;
        background-color: #fff;
        cursor: pointer;
        transition: background-color 0.2s;
      }
      .key:hover {
        background-color: #f0f0f0;
      }
      .key.operator {
        background-color: #dfefff;
      }
      .key.clear {
        background-color: #ffcdd2;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <h1>Máy Tính Giải Tích</h1>
      <input
        type="text"
        id="expression"
        class="expression-input"
        placeholder="Nhập biểu thức (ví dụ: 3x3÷9 hoặc 3^3)"
      />
      <!-- Bàn phím ảo -->
      <div class="keyboard" id="keyboard">
        <!-- Hàng 1 -->
        <div class="key" data-value="7">7</div>
        <div class="key" data-value="8">8</div>
        <div class="key" data-value="9">9</div>
        <div class="key operator" data-value="/">÷</div>
        <!-- Hàng 2 -->
        <div class="key" data-value="4">4</div>
        <div class="key" data-value="5">5</div>
        <div class="key" data-value="6">6</div>
        <div class="key operator" data-value="*">×</div>
        <!-- Hàng 3 -->
        <div class="key" data-value="1">1</div>
        <div class="key" data-value="2">2</div>
        <div class="key" data-value="3">3</div>
        <div class="key operator" data-value="-">–</div>
        <!-- Hàng 4 -->
        <div class="key" data-value="0">0</div>
        <div class="key" data-value=".">.</div>
        <div class="key" data-value="(">(</div>
        <div class="key" data-value=")">)</div>
        <!-- Hàng 5 -->
        <div class="key operator" data-value="√(">√</div>
        <div class="key operator" data-value="^">x<sup>?</sup></div>
        <div class="key operator" data-value="+">+</div>
        <div class="key clear" id="clearKey">C</div>
      </div>
      <button class="btn" id="calculateBtn">Tính Toán</button>
      <div class="result">Kết quả: <span id="result"></span></div>
      <div class="explanation">
        <strong>Giải thích:</strong>
        <div id="explanation"></div>
      </div>
    </div>
    <script>
      /* Hàm chuyển đổi biểu thức gốc sang dạng hiển thị đẹp hơn:
         - Thay "√(" thành "√"
         - Thay "*" (hoặc "x") thành "×"
         - Thay "/" thành "÷"
         - Chuyển a^b thành a<sup>b</sup>
      */
      function formatExprForDisplay(expr) {
        let formatted = expr;
        formatted = formatted.replace(/√\(/g, "√");
        // Nếu người dùng nhập "x" (chữ thường) hoặc "*" thì hiển thị là "×"
        formatted = formatted.replace(/(\*)|x/gi, "×");
        // Chuyển "/" thành "÷"
        formatted = formatted.replace(/\//g, "÷");
        // Chuyển đổi lũy thừa: ví dụ "3^3" thành "3<sup>3</sup>"
        formatted = formatted.replace(/(\d+)\^(\d+)/g, "$1<sup>$2</sup>");
        return formatted;
      }

      // Xử lý nút bàn phím ảo
      const keys = document.querySelectorAll(".key");
      const expressionInput = document.getElementById("expression");

      keys.forEach((key) => {
        key.addEventListener("click", () => {
          const value = key.getAttribute("data-value");
          if (key.classList.contains("clear")) {
            expressionInput.value = "";
          } else {
            expressionInput.value += value;
          }
          expressionInput.focus();
        });
      });

      // Hàm tính toán
      function calculate() {
        let expression = expressionInput.value.trim();
        // Chuyển các ký hiệu hiển thị sang dạng toán học:
        // - Thay "x" (hoặc "×") thành "*" (trừ trường hợp lũy thừa)
        // - Thay "÷" thành "/"
        expression = expression
          .replace(/×/g, "*")
          .replace(/x(?!\^)/g, "*")
          .replace(/÷/g, "/");
        // Chuyển đổi căn: "√(" thành "Math.sqrt("
        expression = expression.replace(/√\(/g, "Math.sqrt(");
        // Chuyển đổi lũy thừa: "^" -> "**"
        expression = expression.replace(/\^/g, "**");

        try {
          const result = eval(expression);
          document.getElementById("result").innerText = result;
          document.getElementById("explanation").innerHTML =
            generateDetailedExplanation(expressionInput.value.trim(), result);
        } catch (error) {
          document.getElementById("result").innerText = "Lỗi!";
          document.getElementById("explanation").innerText =
            "Có lỗi trong quá trình tính toán. Vui lòng kiểm tra lại biểu thức.";
        }
      }

      document
        .getElementById("calculateBtn")
        .addEventListener("click", calculate);
      expressionInput.addEventListener("keydown", function (event) {
        if (event.key === "Enter") {
          calculate();
        }
      });

      // Hàm giải thích chi tiết theo từng trường hợp
      function generateDetailedExplanation(expr, result) {
        let steps = "";
        const exprNoSpace = expr.replace(/\s+/g, "");

        // Tạo biểu thức chuẩn để xác định loại phép toán
        let normalizedExpr = exprNoSpace;
        // Nếu người dùng nhập "x" (nhân) hay "×", chuyển về "*"
        normalizedExpr = normalizedExpr.replace(/x/gi, "*").replace(/×/g, "*");
        // Nếu nhập "÷" chuyển về "/"
        normalizedExpr = normalizedExpr.replace(/÷/g, "/");

        // Chuẩn hóa hiển thị
        let formattedExpr = formatExprForDisplay(expr);

        // Xác định các loại phép toán có trong biểu thức
        const operatorSet = new Set();
        if (normalizedExpr.includes("+")) operatorSet.add("+");
        if (normalizedExpr.includes("-")) operatorSet.add("-");
        if (normalizedExpr.includes("*")) operatorSet.add("*");
        if (normalizedExpr.includes("/")) operatorSet.add("/");
        if (normalizedExpr.includes("^")) operatorSet.add("^");
        if (normalizedExpr.includes("Math.sqrt")) operatorSet.add("√");

        // Nếu biểu thức chỉ chứa các phép nhân và chia (với 2 hoặc nhiều toán tử)
        if (/^[\d\.]+([*/][\d\.]+)+$/.test(normalizedExpr)) {
          let tokens = normalizedExpr.split(/([*/])/);
          if (tokens.length > 3) {
            let stepCount = 1;
            steps += `<strong>Bước ${stepCount}:</strong> Xác định dãy số và phép toán: ${formattedExpr}.<br>`;
            stepCount++;
            let intermediate = Number(tokens[0]);
            for (let i = 1; i < tokens.length; i += 2) {
              let operator = tokens[i];
              let operand = Number(tokens[i + 1]);
              let opDisplay = operator === "*" ? "×" : "÷";
              let newIntermediate =
                operator === "*"
                  ? intermediate * operand
                  : intermediate / operand;
              steps += `<strong>Bước ${stepCount}:</strong> Tính: ${intermediate} ${opDisplay} ${operand} = ${newIntermediate}.<br>`;
              intermediate = newIntermediate;
              stepCount++;
            }
            steps += `<em>Kết quả cuối cùng sau khi tính từ trái sang phải là ${result}.</em>`;
            return steps;
          }
        }

        // Nếu biểu thức có nhiều loại phép toán
        if (operatorSet.size > 1) {
          return `Biểu thức "${formattedExpr}" có nhiều loại phép toán và được tính theo thứ tự ưu tiên: dấu ngoặc () → lũy thừa → căn bậc → nhân/chia (từ trái sang phải) → cộng/trừ. Kết quả cuối cùng là ${result}.`;
        }

        // Nếu chỉ có 1 loại phép toán, xử lý chi tiết từng trường hợp:

        // PHÉP CỘNG
        if (
          normalizedExpr.includes("+") &&
          !/[–\-*\/^√]/.test(normalizedExpr.replace(/\+/g, ""))
        ) {
          let parts = expr.split("+").map((item) => item.trim());
          if (parts.length === 2) {
            steps += `<strong>Bước 1:</strong> Xác định các số cần cộng: ${parts[0]} và ${parts[1]}.<br>`;
            steps += `<strong>Bước 2:</strong> Tính tổng: ${parts[0]} + ${parts[1]} = ${result}.<br>`;
            steps += `<em>Nghĩa là, khi cộng ${parts[0]} và ${parts[1]}, ta được ${result}.</em>`;
          } else if (parts.length > 2) {
            steps += `<strong>Bước 1:</strong> Xác định các số cần cộng: ${parts.join(
              " + "
            )}.<br>`;
            let sum = Number(parts[0]);
            for (let i = 1; i < parts.length; i++) {
              let current = Number(parts[i]);
              let newSum = sum + current;
              steps += `<strong>Bước ${
                i + 1
              }:</strong> Tính: ${sum} + ${current} = ${newSum}.<br>`;
              sum = newSum;
            }
            steps += `<em>Nghĩa là, khi cộng theo thứ tự các số ${parts.join(
              " + "
            )}, ta được ${result}.</em>`;
          }
          return steps;
        }
        // PHÉP TRỪ
        else if (
          normalizedExpr.includes("-") &&
          !/[\+\*\/^√]/.test(normalizedExpr.replace(/\-/g, ""))
        ) {
          let parts = expr.split("-").map((item) => item.trim());
          if (parts.length === 2) {
            steps += `<strong>Bước 1:</strong> Xác định số ban đầu và số cần trừ: ${parts[0]} và ${parts[1]}.<br>`;
            steps += `<strong>Bước 2:</strong> Tính hiệu: ${parts[0]} - ${parts[1]} = ${result}.<br>`;
            steps += `<em>Nghĩa là, khi trừ ${parts[1]} từ ${parts[0]}, ta được ${result}.</em>`;
          } else if (parts.length > 2) {
            steps += `<strong>Bước 1:</strong> Xác định các số cần trừ: ${parts.join(
              " - "
            )}.<br>`;
            let diff = Number(parts[0]);
            for (let i = 1; i < parts.length; i++) {
              let current = Number(parts[i]);
              let newDiff = diff - current;
              steps += `<strong>Bước ${
                i + 1
              }:</strong> Tính: ${diff} - ${current} = ${newDiff}.<br>`;
              diff = newDiff;
            }
            steps += `<em>Kết quả cuối cùng khi trừ theo thứ tự các số ${parts.join(
              " - "
            )}, ta được ${result}.</em>`;
          }
          return steps;
        }
        // PHÉP NHÂN (chỉ chứa "*" hoặc "×")
        else if (
          normalizedExpr.includes("*") &&
          !/[\+\-\/^√]/.test(normalizedExpr.replace(/\*/g, ""))
        ) {
          let parts = expr.split("*").map((item) => item.trim());
          if (parts.length === 2) {
            steps += `<strong>Bước 1:</strong> Xác định các số cần nhân: ${parts[0]} và ${parts[1]}.<br>`;
            steps += `<strong>Bước 2:</strong> Tính tích: ${parts[0]} × ${parts[1]} = ${result}.<br>`;
            steps += `<em>Nghĩa là, khi nhân ${parts[0]} với ${parts[1]}, ta được ${result}.</em>`;
          } else if (parts.length > 2) {
            let allSame = parts.every(
              (item) => Number(item) === Number(parts[0])
            );
            if (allSame) {
              let base = parts[0];
              let exponent = parts.length;
              steps += `<strong>Bước 1:</strong> Xác định các số cần nhân: ${parts.join(
                " × "
              )}.<br>`;
              steps += `<strong>Bước 2:</strong> Nhận thấy tất cả các số đều giống nhau (cơ số = ${base}). Do đó, biểu thức có thể viết dưới dạng lũy thừa: ${base}<sup>${exponent}</sup>.<br>`;
              steps += `<strong>Bước 3:</strong> Tính lũy thừa: ${base}<sup>${exponent}</sup> = ${result}.<br>`;
              steps += `<em>Nghĩa là, khi nhân ${exponent} số ${base} với nhau, ta được ${result}.</em>`;
            } else {
              steps += `<strong>Bước 1:</strong> Xác định các số cần nhân: ${parts.join(
                " × "
              )}.<br>`;
              let intermediate = Number(parts[0]);
              for (let i = 1; i < parts.length; i++) {
                let current = Number(parts[i]);
                let product = intermediate * current;
                steps += `<strong>Bước ${
                  i + 1
                }:</strong> Tính: ${intermediate} × ${current} = ${product}.<br>`;
                intermediate = product;
              }
              steps += `<em>Nghĩa là, khi nhân theo thứ tự các số ${parts.join(
                " × "
              )}, ta được ${result}.</em>`;
            }
          }
          return steps;
        }
        // PHÉP CHIA (chỉ chứa "/" hoặc "÷")
        else if (normalizedExpr.includes("/")) {
          let parts = expr.split("/").map((item) => item.trim());
          if (parts.length === 2) {
            steps += `<strong>Bước 1:</strong> Xác định số bị chia và số chia: ${parts[0]} và ${parts[1]}.<br>`;
            steps += `<strong>Bước 2:</strong> Tính: ${parts[0]} ÷ ${parts[1]} = ${result}.<br>`;
            steps += `<em>Nghĩa là, khi chia ${parts[0]} cho ${parts[1]}, ta được ${result}.</em>`;
          } else if (parts.length > 2) {
            steps += `<strong>Bước 1:</strong> Xác định các số cần chia: ${parts.join(
              " ÷ "
            )}.<br>`;
            let div = Number(parts[0]);
            for (let i = 1; i < parts.length; i++) {
              let current = Number(parts[i]);
              let newDiv = div / current;
              steps += `<strong>Bước ${
                i + 1
              }:</strong> Tính: ${div} ÷ ${current} = ${newDiv}.<br>`;
              div = newDiv;
            }
            steps += `<em>Nghĩa là, khi chia theo thứ tự các số ${parts.join(
              " ÷ "
            )}, ta được ${result}.</em>`;
          }
          return steps;
        }
        // PHÉP LUỶ THỪA (sử dụng "^")
        else if (normalizedExpr.includes("^")) {
          let parts = expr.split("^").map((item) => item.trim());
          steps += `<strong>Bước 1:</strong> Xác định cơ số và số mũ: cơ số = ${parts[0]}, số mũ = ${parts[1]}.<br>`;
          steps += `<strong>Bước 2:</strong> Điều này có nghĩa là nhân ${parts[0]} với chính nó ${parts[1]} lần (tức có ${parts[1]} số ${parts[0]} nhân nhau).<br>`;
          steps += `<strong>Bước 3:</strong> Tính lũy thừa: ${parts[0]}<sup>${parts[1]}</sup> = ${result}.<br>`;
          steps += `<em>Nghĩa là, khi nhân ${parts[0]} với chính nó ${parts[1]} lần, ta được ${result}.</em>`;
          return steps;
        }
        // PHÉP CĂN BẬC HAI
        else if (normalizedExpr.includes("Math.sqrt")) {
          const match = expr.match(/√\(([^)]+)\)/);
          if (match) {
            const num = match[1].trim();
            steps += `<strong>Bước 1:</strong> Xác định số cần lấy căn: ${num}.<br>`;
            steps += `<strong>Bước 2:</strong> Tìm số mà khi nhân với chính nó cho ra ${num} (ví dụ, √${num} = ${result} vì ${result} × ${result} = ${num}).<br>`;
            steps += `<em>Nghĩa là, căn bậc hai của ${num} là ${result}.</em>`;
            return steps;
          }
        }

        return `Biểu thức "${formattedExpr}" được tính theo quy tắc toán học cơ bản và kết quả là ${result}.`;
      }
    </script>
  </body>
</html>
