pipeline {
    agent any

    environment {
        DB_USER = 'root'
        DB_PASSWORD = 'Mqm9kn-Ivbpg*@Pq'  // DB Password
        BASE_BACKUP_DIR = "/home/vroot/DataBaseBackup"
        // DATE = "${new Date().format('dd-MM-yyyy')}"
        // TIME = "${new Date().format('HH-mm-ss')}" // Timestamp for unique filenames
        DATE = sh(script: "date +'%d-%m-%Y'", returnStdout: true).trim()
        TIME = sh(script: "date +'%H:%Mm:%Ss'", returnStdout: true).trim()
      
        BACKUP_DIR = "${BASE_BACKUP_DIR}/${DATE}"
    }

    stages {
        stage('Create Backup Directory') {
            steps {
                script {
                    if (!fileExists(BACKUP_DIR)) {
                        sh "mkdir -p ${BACKUP_DIR}"
                        echo "Backup directory created: ${BACKUP_DIR}"
                    } else {
                        echo "Backup directory already exists: ${BACKUP_DIR}"
                    }
                }
            }
        }

        stage('Perform Backup') {
            steps {
                script {
                    try {
                        echo "Starting database backup..."
                        sh """
                        mysqldump -u ${DB_USER} -p${DB_PASSWORD} vendi_main_tool > "${BACKUP_DIR}/vendi_main_tool.sql"
                        mysqldump -u ${DB_USER} -p${DB_PASSWORD} vendi_admin_tool > "${BACKUP_DIR}/vendi_admin_tool.sql"
                        echo "Database backup completed successfully."
                        """
                    } catch (Exception e) {
                        error "Database backup failed: ${e.message}"
                    }
                }
            }
        }
        stage('Compress SQL Files') {
                    steps {
                        script {
                            echo "Compressing SQL files..."
                            sh """
                            gzip -c "${BACKUP_DIR}/vendi_main_tool.sql" > "${BACKUP_DIR}/${TIME}_vendi_main_tool.sql.gz"
                            gzip -c "${BACKUP_DIR}/vendi_admin_tool.sql" > "${BACKUP_DIR}/${TIME}_vendi_admin_tool.sql.gz"
                            echo "SQL files compressed successfully."
                            """
                        }
                    }
                }        

        stage('Cleanup Old Backups') {
            steps {
                script {
                    echo "Cleaning up old backups..."
                    sh """
                    find "${BASE_BACKUP_DIR}" -name "*.gz" -type f -mtime +7 -exec rm -rf {} \\;
                    find "${BASE_BACKUP_DIR}" -mindepth 1 -type d -mtime +7 -exec rm -rf {} \\;
                    echo "Old backups cleaned up successfully."
                    """
                }
            }
        }
    }
}



