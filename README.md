<!DOCTYPE html>

<html>

<head>

    <title>Bảng điều khiển Giám sát Lúa</title>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/paho-mqtt/1.0.1/mqttws31.js" type="text/javascript"></script>

    <style>

        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; margin: 20px; background-color: #f7f9fc; }

        .card { background-color: white; padding: 20px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.08); margin-bottom: 20px; }

        h1, h2 { color: #1a2533; border-bottom: 2px solid #eef1f5; padding-bottom: 10px; }

        #temp-value, #humi-value { font-size: 2.5em; font-weight: bold; color: #007BFF; }

        #disease-info { font-size: 1.2em; color: #DC3545; min-height: 50px; font-weight: bold;}

        #status { font-size: 0.9em; color: #6c757d; }

    </style>

</head>

<body>

    <h1>Bảng điều khiển Giám sát Lúa - Trạng thái trực tiếp</h1>

    <p id="status">Đang kết nối đến Broker...</p>



    <div class="card">

        <h2>Nhiệt độ Đất</h2>

        <p><span id="temp-value">--</span> °C</p>

    </div>



    <div class="card">

        <h2>Độ ẩm Đất</h2>

        <p><span id="humi-value">--</span> %</p>

    </div>



    <div class="card">

        <h2>Thông tin Bệnh</h2>

        <p id="disease-info">Chưa có thông tin</p>

    </div>



    <script>

        // ================== CẤU HÌNH KẾT NỐI CỦA BẠN ==================

        var brokerHost = "75330e23591346938c002c40c2dce261.s1.eu.hivemq.cloud"; // <-- Dán URL Broker của bạn vào đây

        var brokerPort = 8884; // Port WebSocket an toàn (thường là 8884 cho HiveMQ Cloud)

        var mqttUser = "TEN_USER_CUA_BAN";     // <-- Điền Username HiveMQ bạn đã tạo

        var mqttPass = "MAT_KHAU_CUA_BAN";     // <-- Điền Password HiveMQ bạn đã tạo

        

        var sensorTopic = "DuAnLua/sensors";   // Kênh (topic) lắng nghe dữ liệu cảm biến

        var diseaseTopic = "DuAnLua/disease";  // Kênh (topic) lắng nghe thông tin bệnh

        // =============================================================



        var clientID = "WebAppClient_" + parseInt(Math.random() * 1000);



        // Tạo một đối tượng client MQTT mới

        var client = new Paho.MQTT.Client(brokerHost, brokerPort, clientID);



        // Đặt các hàm callback

        client.onConnectionLost = onConnectionLost;

        client.onMessageArrived = onMessageArrived;

        

        // Cấu hình thông tin kết nối

        var options = {

            onSuccess: onConnect,

            onFailure: onConnectionLost,

            userName: mqttUser,

            password: mqttPass,

            useSSL: true // Bắt buộc dùng kết nối an toàn với HiveMQ Cloud

        };



        // Bắt đầu kết nối

        client.connect(options);



        // Hàm này sẽ được gọi khi kết nối thành công

        function onConnect() {

            console.log("Đã kết nối MQTT!");

            document.getElementById("status").textContent = "Đã kết nối! Đang chờ dữ liệu...";

            // Đăng ký (subscribe) để lắng nghe tin nhắn từ các kênh

            client.subscribe(sensorTopic);

            client.subscribe(diseaseTopic);

        }



        // Hàm này được gọi khi mất kết nối

        function onConnectionLost(responseObject) {

            if (responseObject.errorCode !== 0) {

                document.getElementById("status").textContent = "Mất kết nối! Đang thử lại...";

                console.log("Mất kết nối: " + responseObject.errorMessage);

            }

        }



        // Hàm này được gọi mỗi khi có tin nhắn mới đến

        function onMessageArrived(message) {

            // Cập nhật thông tin bệnh

            if (message.destinationName === diseaseTopic) {

                document.getElementById("disease-info").textContent = message.payloadString;

            }



            // Cập nhật thông số cảm biến

            if (message.destinationName === sensorTopic) {

                var payload = message.payloadString;

                // Kiểm tra xem payload có phải là chuỗi key=value không

                if(payload.includes("=")){

                    var params = new URLSearchParams(payload);

                    var temp = params.get('t'); // Lấy giá trị của 't' (nhiệt độ)

                    var humi = params.get('h'); // Lấy giá trị của 'h' (độ ẩm)

                    if(temp) document.getElementById("temp-value").textContent = parseFloat(temp).toFixed(1);

                    if(humi) document.getElementById("humi-value").textContent = parseFloat(humi).toFixed(0);

                } else { // Nếu là chuỗi lỗi thì hiển thị ra

                    document.getElementById("temp-value").textContent = payload;

                    document.getElementById("humi-value").textContent = payload;

                }

            }

        }

    </script>

</body>

</html>
