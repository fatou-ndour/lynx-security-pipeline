pipeline {

    agent any

    triggers {
        githubPush()
    }

    environment {
        REPO_URL    = "https://github.com/Clivern/Lynx.git"
        MAIL_TO     = "fatoundour@esp.sn"
        REPORT_DIR  = "${WORKSPACE}\\reports"
    }

    options {
        timeout(time: 60, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    stages {

        // ── TÂCHE 1 : Cloner le projet Lynx (Terraform Backend)
        stage('Checkout Lynx') {
            steps {
                echo '=== Clonage du projet Lynx (Terraform Backend - Elixir) ==='
                git url: "${REPO_URL}", branch: 'main'
                bat "if not exist \"%REPORT_DIR%\" mkdir \"%REPORT_DIR%\""
                echo 'Lynx clone avec succes'
            }
        }

        // ── TÂCHE 2 : SAST avec Sobelow (outil OWASP pour Elixir/Phoenix)
        // Sobelow est l'outil recommande par OWASP pour les projets Phoenix
        // Source : https://owasp.org/www-community/Source_Code_Analysis_Tools
        // Detecte : XSS, SQLi, traversee de chemins, configs non securisees
        stage('SAST - Sobelow (OWASP)') {
            steps {
                echo '=== SAST : Sobelow - Outil OWASP pour Phoenix/Elixir ==='
                script {
                    bat """
                        docker run --rm ^
                          -v "%WORKSPACE%:/app" ^
                          nifty/sobelow:latest ^
                          --format json ^
                          --out /app/reports/sobelow-report.json ^
                          --exit ^
                          /app
                    """
                }
                echo 'SAST Sobelow termine - rapport : reports/sobelow-report.json'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'reports/sobelow-report.json',
                                     allowEmptyArchive: true
                }
            }
        }

    }

    // ── TÂCHE 3 : Rapport par email
    post {
        always {
            archiveArtifacts artifacts: 'reports/**/*',
                             allowEmptyArchive: true

            echo '=== Envoi du rapport par email ==='

            emailext(
                subject: "[Jenkins] Lynx SAST Scan - Build #${BUILD_NUMBER} - ${currentBuild.result ?: 'RUNNING'}",
                to: "${MAIL_TO}",
                mimeType: 'text/html',
                attachmentsPattern: 'reports/sobelow-report.json',

                body: """
<html>
<body style="margin:0;padding:0;background:#f0f2f5;font-family:Arial,sans-serif;">
<table width="100%" cellpadding="0" cellspacing="0">
<tr><td align="center" style="padding:30px 16px;">
<table width="620" cellpadding="0" cellspacing="0"
       style="background:#fff;border-radius:8px;
              box-shadow:0 2px 8px rgba(0,0,0,0.12);overflow:hidden;">

  <tr>
    <td style="background:#1e2a3a;padding:22px 28px;">
      <div style="color:#fff;font-size:20px;font-weight:bold;">
        Rapport SAST - Lynx Security Scan
      </div>
      <div style="color:#7a8fa6;font-size:13px;margin-top:4px;">
        Pipeline CI/CD Jenkins - Outil OWASP Sobelow
      </div>
    </td>
  </tr>

  <tr>
    <td style="background:${currentBuild.result == 'SUCCESS' ? '#1a7f37' : '#842029'};
               padding:13px 28px;text-align:center;">
      <span style="color:#fff;font-weight:bold;font-size:15px;">
        ${currentBuild.result == 'SUCCESS' ? 'BUILD REUSSI - Aucune vulnerabilite critique' : 'BUILD ECHOUE - Vulnerabilites detectees'}
      </span>
    </td>
  </tr>

  <tr><td style="padding:20px 28px 10px;">
    <table width="100%" style="font-size:13px;border-collapse:collapse;">
      <tr style="background:#f8f9fa;">
        <td style="padding:8px 12px;color:#666;">Job</td>
        <td style="padding:8px 12px;font-weight:bold;">${JOB_NAME}</td>
        <td style="padding:8px 12px;color:#666;">Build</td>
        <td style="padding:8px 12px;font-weight:bold;">#${BUILD_NUMBER}</td>
      </tr>
      <tr>
        <td style="padding:8px 12px;color:#666;">Projet</td>
        <td style="padding:8px 12px;" colspan="3">Clivern/Lynx (Terraform Backend - Elixir)</td>
      </tr>
      <tr style="background:#f8f9fa;">
        <td style="padding:8px 12px;color:#666;">Commit</td>
        <td style="padding:8px 12px;font-family:monospace;" colspan="3">
          ${env.GIT_COMMIT?.take(8) ?: 'N/A'} sur ${env.GIT_BRANCH ?: 'main'}
        </td>
      </tr>
    </table>
  </td></tr>

  <tr><td style="padding:10px 28px 20px;">
    <div style="font-size:14px;font-weight:bold;color:#1e2a3a;
                border-bottom:2px solid #e9ecef;padding-bottom:8px;margin-bottom:12px;">
      Resultat du scan SAST
    </div>

    <div style="background:#e8f4fd;border-left:4px solid #1a73e8;
                padding:12px 16px;border-radius:0 4px 4px 0;margin-bottom:10px;">
      <strong>SAST - Sobelow (OWASP)</strong><br>
      <span style="font-size:13px;color:#444;">
        Analyse statique du code Elixir/Phoenix<br>
        Detecte : XSS, SQLi, traversee de chemins, configs non securisees<br>
        Source OWASP : https://owasp.org/www-community/Source_Code_Analysis_Tools<br>
        Rapport joint : sobelow-report.json
      </span>
    </div>
  </td></tr>

  <tr><td style="padding:0 28px 24px;text-align:center;">
    <a href="${BUILD_URL}"
       style="background:#1e2a3a;color:#fff;padding:11px 28px;
              text-decoration:none;border-radius:5px;
              font-size:14px;font-weight:bold;">
      Voir le build complet sur Jenkins
    </a>
  </td></tr>

  <tr><td style="background:#f8f9fa;padding:12px 28px;
                 border-top:1px solid #e9ecef;font-size:11px;color:#999;">
    Genere automatiquement par Jenkins apres chaque git push
  </td></tr>

</table>
</td></tr>
</table>
</body>
</html>
                """
            )
        }
        success { echo 'Aucune vulnerabilite critique detectee' }
        failure { echo 'Vulnerabilites detectees - voir le rapport' }
    }
}
