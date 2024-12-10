pipeline {
    agent any

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
                conda run -n audiogpt pip install --upgrade pip

                if [ -f requirements.txt ]; then
                    conda run -n audiogpt pip install -r requirements.txt
                fi
                '''
            }
        }
        stage('Upgrade Dependencies') {
            steps {
                sh '''
                export PATH="$MINICONDA_DIR/bin:$PATH"
                conda run -n audiogpt pip install --upgrade pip
                conda run -n audiogpt pip install --upgrade diffusers transformers safetensors huggingface_hub
                '''
            }
        }


        // stage('Download Models') {
        //     steps {
        //         sh '''
        //         export PATH="$MINICONDA_DIR/bin:$PATH"
        //         conda run -n audiogpt bash download.sh
        //         '''
        //     }
        // }

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
            echo "Build and run succeeded!"
        }
        failure {
            echo "Build failed. Check logs."
        }
    }
}
