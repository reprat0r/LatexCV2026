## FLOWCHART test


'''mermaid
graph TD
    %% Define Styles
    classDef hardware fill:#f9f,stroke:#333,stroke-width:2px;
    classDef preprocessing fill:#bbf,stroke:#333,stroke-width:2px;
    classDef training fill:#fbf,stroke:#333,stroke-width:2px;
    classDef offline fill:#fff,stroke:#333,stroke-width:1px,stroke-dasharray: 5 5;

    subgraph Data_Collection ["1. Data Acquisition (Field Test / Drone)"]
        A[3 Synchronized Shutter Cameras] --> B1[Camera 1: RGB Image]
        A --> B2[Camera 2: UV Image]
        A --> B3[Camera 3: NIR Image]
    end

    subgraph Preprocessing ["2. Ground Station Preprocessing Pipeline (OpenCV)"]
        B1 & B2 & B3 --> C[Load Calibration Chesboard Target Data]
        C --> D[Calculate SIFT / ORB Feature Matching]
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
        I -- Small Dataset < 1,000 images --> J[Configure YOLOv8x via YAML: set ch=5]
        J --> K[Dataset Preparation]
        K --> K1[Apply 360 Rotation & Flipping]
        K --> K2[Downscale Close-ups to >= 32x32 px]
        K --> K3[Apply Mosaic Tiling on Backgrounds]
        K1 & K2 & K3 --> L[Train YOLOv8x Model from Scratch or Partial Weights]
        
        %% Path B: Large Dataset (RT-DETRv2)
        I -- Large Dataset > 5,000 images --> M[Configure RT-DETRv2 Transformer in Ultralytics]
        M --> N[Format Target Labels into COCO JSON Format]
        N --> O[Train RT-DETRv2 Model utilizing Shape Attention]
    end

    subgraph Evaluation_Deployment ["4. Offline Production Inference Pipeline"]
        L & O --> P[Post-Flight High-Res Imagery Upload]
        P --> Q[SAHI Wrapper: Slice 5-Channel Image into Grids]
        Q --> R[Run Model Inference on patches]
        R --> S[Non-Maximum Suppression NMS: Stitch Bounding Boxes]
        S --> T[Final Bird Detection Log: GPS & Species Coordinates]
    end

    %% Apply Styles
    class A,B1,B2,B3 hardware;
    class C,D,E,F1,F2,G,H preprocessing;
    class J,K,L,M,N,O training;
    class P,Q,R,S,T offline;
'''
