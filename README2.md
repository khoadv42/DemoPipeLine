pipeline {
    agent any
    tools {
        maven 'M3'
        jdk 'JDK17'
    }
    stages {
        // Stage 1: Checkout mã nguồn
        stage('Checkout Secondary Repo') {
            steps {
                git branch: 'main', 
                     credentialsId: 'DemoPipeLine', 
                     url: 'https://github.com/khoadv42/DemoPipeLine.git'
            }
        }
        
        // Stage 2: Build và Đóng gói
        stage('Build and Test') {
            steps {
                // Đổi 'mvn clean package' thành 'mvn clean install'
                // nếu bạn muốn chạy Unit Test. 'package' sẽ không chạy test nếu bạn không config.
                sh 'mvn clean install' 
            }
        }
        
        // Stage 3: Chạy Service
        stage('Run Service') {
            steps {
                script {
                    def jarName = 'demo-0.0.1-SNAPSHOT.jar' 
                    // Chạy ngầm (nohup ... &)
                    sh "nohup java -jar target/${jarName} > app.log 2>&1 &" 
                }
                
                sh 'sleep 10' 
                // Kiểm tra service (sẽ thất bại nếu service không chạy)
                sh 'curl -f http://localhost:8080/actuator/health'
            }
        }
    }
    
    // --- KHẮC PHỤC LỖI LOGIC CLEAN UP ---
    // Khối 'post' đảm bảo dọn dẹp chạy sau khi Build/Test/Run hoàn tất
    post {
        // Luôn luôn chạy (thành công hay thất bại)
        always {
            stage('Clean Up') { 
                steps {
                    // Dùng || true để lệnh không fail nếu không tìm thấy tiến trình
                    echo 'Stopping application service...'
                    sh 'pkill -f "java -jar" || true'
                }
            }
        }
    }
}