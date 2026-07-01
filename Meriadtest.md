# Project Workflow: Offline Multi-Spectral Bird Detection

Below is the complete architectural workflow for processing the 5-channel early fusion image data.

```mermaid
graph LR
    %% Define Strict High-Contrast Black & White Styles
    classDef bwStyle fill:#ffffff,stroke:#000000,stroke-width:2px,color:#000000;
    classDef dashedStyle fill:#ffffff,stroke:#000000,stroke-width:2px,stroke-dasharray: 5 5,color:#000000;

    subgraph Data_Collection ["1. Data Acquisition (Field Test / Drone)"]
        A[3 Synchronized Shutter Cameras] --> B1[Camera 1: RGB Image]
        A --> B2[Camera 2: UV Image]
        A --> B3[Camera 3: NIR Image]
    end

    subgraph Preprocessing ["2. Ground Station Preprocessing Pipeline (OpenCV)"]
        B1 --> C[Load Calibration Chessboard Data]
        B2 --> C
        B3 --> C
        C --> D[Calculate SIFT or ORB Feature Matching]
        D --> E[Generate Homography Matrix H]
        
        E --> F1[Warp UV Image to RGB Master]
        E --> F2[Warp NIR Image to RGB Master]
        
        B1 --> G[Stack Channels together]
        F1 --> G
        F2 --> G
        G --> H[Output 5-Channel Tensor Array: Shape H, W, 5]
    end

    subgraph Training_Phase ["3. Neural Network Pipeline (Offline Server)"]
        H --> I{Dataset Size?}
        
        %% Path A: Small Dataset (YOLOv8x)
        I -- Less than 1000 images --> J[Configure YOLOv8x via YAML: set ch=5]
        J --> K[Dataset Preparation]
        K --> K1[Apply 360 Rotation and Flipping]
        K --> K2[Downscale Close-ups to minimum 32x32 px]
        K --> K3[Apply Mosaic Tiling on Backgrounds]
        K1 --> L[Train YOLOv8x Model from Scratch]
        K2 --> L
        K3 --> L
        
        %% Path B: Large Dataset (RT-DETRv2)
        I -- More than 5000 images --> M[Configure RT-DETRv2 Transformer]
        M --> N[Format Target Labels into COCO JSON]
        N --> O[Train RT-DETRv2 Model utilizing Shape Attention]
    end

    subgraph Evaluation_Deployment ["4. Offline Production Inference Pipeline"]
        L --> P[Post-Flight High-Res Imagery Upload]
        O --> P
        P --> Q[SAHI Wrapper: Slice 5-Channel Image into Grids]
        Q --> R[Run Model Inference on patches]
        R --> S[Non-Maximum Suppression NMS: Stitch Boxes]
        S --> T[Final Bird Detection Log: GPS Coordinates]
    end

    %% Apply Black and White Styles to all Nodes
    class A,B1,B2,B3,C,D,E,F1,F2,G,H,I,J,K,K1,K2,K3,L,M,N,O bwStyle;
    class P,Q,R,S,T dashedStyle;
