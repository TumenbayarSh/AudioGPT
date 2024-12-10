pipeline {
    agent {
        label 'linux'
    }

    environment {
        MINICONDA_DIR = "${WORKSPACE}/miniconda"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Miniconda') {
            steps {
                sh '''
                # Download and install Miniconda if not installed
                if [ ! -d "$MINICONDA_DIR" ]; then
                    echo "Installing Miniconda..."
                    wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O miniconda.sh
                    bash miniconda.sh -b -p $MINICONDA_DIR
                else
                    echo "Miniconda already installed."
                fi
                '''
            }
        }

        stage('Set PATH for Conda') {
            steps {
                sh '''
                # Add Miniconda to the PATH for this build
                export PATH="$MINICONDA_DIR/bin:$PATH"
                conda --version
                '''
            }
        }

        stage('Set up Conda Environment') {
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                # Create the audiogpt environment if it does not exist
                if ! conda env list | grep audiogpt; then
                    conda create -y -n audiogpt python=3.8
                fi
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                # Upgrade pip and install lint/test tools
                conda run -n audiogpt pip install --upgrade pip
                conda run -n audiogpt pip install flake8 pytest

                # Install project requirements
                if [ -f requirements.txt ]; then
                    conda run -n audiogpt pip install -r requirements.txt
                fi
                '''
            }
        }

        stage('Lint') {
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                conda run -n audiogpt flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics
                # Non-blocking lint run:
                conda run -n audiogpt flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                conda run -n audiogpt pytest
                '''
            }
        }

        stage('Download Models') {
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                conda run -n audiogpt bash download.sh
                '''
            }
        }

        stage('Run AudioGPT') {
            environment {
                OPENAI_API_KEY = credentials('OPENAI_API_KEY')
            }
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                export OPENAI_API_KEY="${OPENAI_API_KEY}"
                conda run -n audiogpt python audio-chatgpt.py
                '''
            }
        }
    }

    post {
        always {
            echo "Pipeline run completed."
        }
        success {
            echo "Build, test, and run succeeded!"
        }
        failure {
            echo "Build or tests failed. Check the logs above."
        }
    }
}