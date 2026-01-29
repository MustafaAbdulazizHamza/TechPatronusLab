# Scapyfy Project
=== "Description"
    **Scapyfy** is an AI-powered network security toolkit that combines LLM intelligence with powerful packet crafting capabilities. It provides both an **LLM agent** for automated network analysis and **direct tool access** for manual operations. The platform supports **multiple LLM providers** (OpenAI, Google Gemini, Anthropic Claude, Ollama), features a modern **web interface**, and exposes a **REST API** secured with **JWT authentication** and **TLS** support.





=== "Code"
    ```python linenums="1" title="main.py"
    from contextlib import asynccontextmanager
    from fastapi import FastAPI, Request
    from fastapi.middleware.cors import CORSMiddleware
    from fastapi.staticfiles import StaticFiles
    from fastapi.responses import FileResponse
    from database import get_db, engine
    from models import Base, User
    from hashing import hash_password
    from routers import user, login, crafter, tools
    from logger import get_logger
    import os
    import secrets
    import time
    from pathlib import Path

    Base.metadata.create_all(bind=engine)

    logger = get_logger()


    @asynccontextmanager
    async def lifespan(app: FastAPI):
        await initialize_admin_user()
        yield


    async def initialize_admin_user():
        db = next(get_db())
        try:
            admin_user = db.query(User).filter(User.id == 0).first()
            
            if not admin_user:
                admin_password = os.getenv("ROOT_PASSWORD")
                if not admin_password:
                    admin_password = secrets.token_urlsafe(16)
                    print("=" * 60)
                    print("🔐 ROOT USER CREATED")
                    print("=" * 60)
                    print(f"   Username: root")
                    print(f"   Password: {admin_password}")
                    print("=" * 60)
                    print("⚠️  SECURITY WARNING:")
                    print("   Save this password securely - it will not be shown again!")
                    print("   Set ROOT_PASSWORD env variable to use a custom password.")
                    print("=" * 60)
                else:
                    print("✅ Root user created with password from ROOT_PASSWORD env var")
                
                admin_user = User(
                    id=0,
                    username="root",
                    email=os.getenv("ROOT_EMAIL", "root@scapyfy.example.com"),
                    hashed_password=hash_password(admin_password),
                    is_active=True
                )
                db.add(admin_user)
                db.commit()
            else:
                print("ℹ️  Root user already exists")
        except Exception as e:
            print(f"❌ Error initializing root user: {e}")
            db.rollback()
        finally:
            db.close()


    app = FastAPI(
        title="Scapyfy",
        description="""
        🧙‍♂️ Scapyfy - AI-powered Network Security Toolkit

        A secure LLM agent that performs packet crafting and network analysis tasks.

        ## Features
        - Packet Crafting: Craft and send packets, receive analysis reports
        - Multiple LLM Providers**: OpenAI, Google Gemini, Anthropic Claude, Ollama
        - Network Tools: NMAP, Traceroute, Ping, Hping3, Port Scanning

        ## Authentication
        All endpoints require JWT Bearer token authentication.
        Use `/auth/login` to obtain a token.
        """,
        version="2.0.0",
        lifespan=lifespan
    )

    ALLOWED_ORIGINS = os.getenv("CORS_ORIGINS", "*").split(",")
    app.add_middleware(
        CORSMiddleware,
        allow_origins=ALLOWED_ORIGINS if ALLOWED_ORIGINS != ["*"] else ["*"],
        allow_credentials=True,
        allow_methods=["GET", "POST", "PUT", "DELETE"],
        allow_headers=["*"],
    )


    @app.middleware("http")
    async def log_requests(request: Request, call_next):
        start_time = time.time()
        
        response = await call_next(request)
        
        duration_ms = (time.time() - start_time) * 1000
        
        path = request.url.path
        if not path.startswith(("/assets", "/styles.css", "/app.js", "/docs", "/openapi.json", "/redoc")):
            user = "anonymous"
            if hasattr(request.state, "user"):
                user = getattr(request.state.user, "username", "anonymous")
            
            logger.log_api_request(
                user=user,
                method=request.method,
                path=path,
                status_code=response.status_code,
                duration_ms=duration_ms,
                client_ip=request.client.host if request.client else None
            )
        
        return response


    app.include_router(login.router)
    app.include_router(user.router)
    app.include_router(crafter.router)
    app.include_router(tools.router)

    FRONTEND_DIR = os.path.join(os.path.dirname(__file__), "frontend")

    if os.path.exists(FRONTEND_DIR):
        app.mount("/assets", StaticFiles(directory=os.path.join(FRONTEND_DIR, "assets")), name="assets")
        
        @app.get("/styles.css")
        async def get_styles():
            response = FileResponse(os.path.join(FRONTEND_DIR, "styles.css"), media_type="text/css")
            response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
            response.headers["Pragma"] = "no-cache"
            response.headers["Expires"] = "0"
            return response
        
        @app.get("/app.js")
        async def get_app_js():
            response = FileResponse(os.path.join(FRONTEND_DIR, "app.js"), media_type="application/javascript")
            response.headers["Cache-Control"] = "no-cache, no-store, must-revalidate"
            response.headers["Pragma"] = "no-cache"
            response.headers["Expires"] = "0"
            return response


    @app.get("/")
    async def root():
        index_path = os.path.join(FRONTEND_DIR, "index.html")
        if os.path.exists(index_path):
            return FileResponse(index_path, media_type="text/html")
        
        return {
            "message": "🧙‍♂️ Scapyfy - AI-powered Network Security Toolkit",
            "version": "2.0.0",
            "docs": "/docs",
            "redoc": "/redoc",
            "status": "running",
            "features": [
                "Multi-LLM support (OpenAI, Gemini, Claude, Ollama)",
                "AI-powered packet crafting with detailed reports",
                "Network scanning (NMAP, port scan, ARP)",
                "Network diagnostics (ping, traceroute, hping3)"
            ]
        }


    @app.get("/api")
    def api_info():
        return {
            "message": "🧙‍♂️ Scapyfy API",
            "version": "2.0.0",
            "docs": "/docs",
            "status": "running"
        }


    @app.get("/health")
    def health_check():
        return {
            "status": "healthy",
            "version": "2.0.0"
        }


    if __name__ == "__main__":
        import uvicorn
        import argparse
        
        parser = argparse.ArgumentParser(description="Scapyfy - AI-Powered Packet Crafter")
        parser.add_argument("--host", default=os.getenv("HOST", "0.0.0.0"), help="Host to bind to")
        parser.add_argument("--port", type=int, default=int(os.getenv("PORT", "8000")), help="Port to bind to")
        parser.add_argument("--reload", action="store_true", default=os.getenv("RELOAD", "false").lower() == "true", help="Enable auto-reload")
        parser.add_argument("--ssl-certfile", default=os.getenv("SSL_CERTFILE"), help="Path to SSL certificate file")
        parser.add_argument("--ssl-keyfile", default=os.getenv("SSL_KEYFILE"), help="Path to SSL key file")
        parser.add_argument("--ssl-ca-certs", default=os.getenv("SSL_CA_CERTS"), help="Path to CA certificates for client verification")
        parser.add_argument("--ssl-cert-reqs", type=int, default=int(os.getenv("SSL_CERT_REQS", "0")), 
                            help="Client certificate requirement: 0=none, 1=optional, 2=required")
        
        args = parser.parse_args()
        
        uvicorn_kwargs = {
            "host": args.host,
            "port": args.port,
            "reload": args.reload,
        }
        
        if args.ssl_certfile and args.ssl_keyfile:
            if not Path(args.ssl_certfile).exists():
                print(f"❌ SSL certificate file not found: {args.ssl_certfile}")
                exit(1)
            if not Path(args.ssl_keyfile).exists():
                print(f"❌ SSL key file not found: {args.ssl_keyfile}")
                exit(1)
                
            uvicorn_kwargs["ssl_certfile"] = args.ssl_certfile
            uvicorn_kwargs["ssl_keyfile"] = args.ssl_keyfile
            
            if args.ssl_ca_certs:
                if not Path(args.ssl_ca_certs).exists():
                    print(f"❌ CA certificates file not found: {args.ssl_ca_certs}")
                    exit(1)
                uvicorn_kwargs["ssl_ca_certs"] = args.ssl_ca_certs
                uvicorn_kwargs["ssl_cert_reqs"] = args.ssl_cert_reqs
                
                cert_req_names = {0: "CERT_NONE", 1: "CERT_OPTIONAL", 2: "CERT_REQUIRED"}
                print(f"🔐 Client certificate verification: {cert_req_names.get(args.ssl_cert_reqs, 'UNKNOWN')}")
            
            print(f"🔒 Starting with TLS enabled")
            print(f"📡 Server: https://{args.host}:{args.port}")
        else:
            print(f"🔓 Starting without TLS (development mode)")
            print(f"📡 Server: http://{args.host}:{args.port}")
        
        print(f"📚 API Docs: {'https' if args.ssl_certfile else 'http'}://{args.host}:{args.port}/docs")
        
        uvicorn.run("main:app", **uvicorn_kwargs)
    ```
=== "Documentation"
    ## 🧙‍♂️ Scapyfy 2.0.0

    ![](https://github.com/MustafaAbdulazizHamza/Scapyfy/blob/main/scapyfy.png?raw=true)
    
    ---
    
    **Scapyfy** is an AI-powered network security toolkit that combines LLM intelligence with powerful packet crafting capabilities. It provides both an **LLM agent** for automated network analysis and **direct tool access** for manual operations. The platform supports **multiple LLM providers** (OpenAI, Google Gemini, Anthropic Claude, Ollama), features a modern **web interface**, and exposes a **REST API** secured with **JWT authentication** and **TLS** support.


    ### Features
    ---
    ### AI-Powered Network Tools
    - **LLM Agent** - Automated network analysis and reporting
    - **AI-assistant** - Interactive chat interface for network analysis in direct tool access mode.
    
    ### Multi-LLM Support
    - **OpenAI** (GPT-3.5, GPT-4)
    - **Google Gemini** (Gemini 1.5 Flash/Pro)
    - **Anthropic Claude** (Claude 3.5 Sonnet)
    - **Ollama** (Local)

    #### Network Tools
    - **Packet Crafting** (Scapy) - Custom TCP/IP/UDP/ICMP packets
    - **Nmap Scanner** - Port scanning, service detection, OS fingerprinting
    - **Traceroute** - Network path discovery
    - **Ping** - Host reachability testing
    - **Hping3** - Advanced packet probing
    - **Port Scanner** - Fast Scapy-based scanning
    - **ARP Scanner** - Local network discovery

    ### Requirements
    ---

    To run Scapyfy, you will need the following:

    1. A **Linux machine** (desktop, server, etc.)
    2. **Superuser privileges** (`sudo`) for packet crafting
    3. **Python 3.10+**
    4. At least one **LLM API Key** (OpenAI, Google, or Anthropic) OR a running **Ollama** instance

    #### System Dependencies
    ```bash
    # For Nmap scanning
    sudo apt install nmap

    # For Hping3 
    sudo apt install hping3

    # For traceroute
    sudo apt install traceroute
    ```


    ### Installation
    ---

    Follow these steps to get Scapyfy set up:

    1. **Clone the repository:**
    ```bash
    git clone https://github.com/MustafaAbdulazizHamza/Scapyfy.git
    cd Scapyfy
    ```

    2. **Create and activate virtual environment:**
    ```bash
    python3 -m venv env
    source env/bin/activate
    ```

    3. **Install dependencies:**
    ```bash
    pip install -r requirements.txt
    ```

    4. **Configure environment variables:**
    Create a `.env` file based on the configuration specified in `env.example` file. 


    ### Execution
    ---

    Since packet crafting requires low-level access, run with **superuser privileges**:

    #### Without TLS
    ```bash
    sudo bash execute.sh
    ```

    #### With TLS
    ```bash
    # Using your own certificates
    sudo bash execute.sh --ssl-cert /path/to/server.crt --ssl-key /path/to/server.key
    ```

    ### Notes
    ---

    1. **API Documentation**: Available at `/docs` (Swagger UI) and `/redoc` (ReDoc) after starting the server
    2. **Logs**: Execution logs are stored in `./logs/scapyfy_executions.log`
    3. **Security**: Always change the generated root password, set a strong `SECRET_KEY`, and run the server on a dedicated virtual machine in production.



    ## Disclaimer
    ---

    - This project is designed for **educational purposes** and authorized security testing only.

    - **Always ensure you have proper authorization before performing network scans or packet injection.**


=== "Download"
    [![GitHub](https://img.shields.io/badge/Source-GitHub-blue)](https://github.com/MustafaAbdulazizHamza/Scapyfy)

