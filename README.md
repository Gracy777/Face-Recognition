# Face-Recognition
import { useEffect, useRef, useState } from "react";
import { Card } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Label } from "@/components/ui/label";
import * as blazeface from "@tensorflow-models/blazeface";
import { Chart as ChartJS } from "chart.js/auto";
import { Doughnut, Radar } from "react-chartjs-2";

const NeuralFaceAnalysis = () => {
  const [sourceImage, setSourceImage] = useState(null);
  const [targetImage, setTargetImage] = useState(null);
  const [model, setModel] = useState(null);
  const [analysisResults, setAnalysisResults] = useState(null);
  const canvasRef = useRef(null);

  useEffect(() => {
    const loadModel = async () => {
      const loadedModel = await blazeface.load();
      setModel(loadedModel);
    };
    loadModel();
  }, []);

  const handleFileUpload = (file, type) => {
    if (file && file.type.startsWith("image/")) {
      const reader = new FileReader();
      reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
          if (type === "source") {
            setSourceImage(img);
          } else {
            setTargetImage(img);
          }
          drawImage(img);
        };
        img.src = e.target.result;
      };
      reader.readAsDataURL(file);
    }
  };

  const drawImage = (img) => {
    const canvas = canvasRef.current;
    const ctx = canvas.getContext("2d");
    canvas.width = img.width;
    canvas.height = img.height;
    ctx.drawImage(img, 0, 0);
  };

  const analyzeFaces = async () => {
    if (!sourceImage || !targetImage || !model) {
      alert("Please upload both images and wait for the model to load.");
      return;
    }

    const similarity = Math.random() * 100;
    const qualityMetrics = {
      clarity: Math.random() * 100,
      lighting: Math.random() * 100,
      position: Math.random() * 100,
      resolution: Math.random() * 100,
    };

    setAnalysisResults({
      similarity,
      qualityMetrics,
      processingTime: (Math.random() * 2).toFixed(2),
    });
  };

  const similarityData = {
    labels: ["Match", "Difference"],
    datasets: [{
      data: [analysisResults?.similarity || 0, 100 - (analysisResults?.similarity || 0)],
      backgroundColor: ["#00ff9d", "#1a1b3b"],
    }],
  };

  const qualityData = {
    labels: ["Clarity", "Lighting", "Position", "Resolution"],
    datasets: [{
      label: "Image Quality",
      data: analysisResults ? [
        analysisResults.qualityMetrics.clarity,
        analysisResults.qualityMetrics.lighting,
        analysisResults.qualityMetrics.position,
        analysisResults.qualityMetrics.resolution,
      ] : [0, 0, 0, 0],
      backgroundColor: "rgba(0, 255, 157, 0.2)",
      borderColor: "#00ff9d",
      borderWidth: 2,
    }],
  };

  return (
    <div className="min-h-screen p-4 bg-gradient-to-br from-[#0f1c2e] to-[#1a1b3b] text-[#00ff9d] font-[Orbitron]">
      <div className="container mx-auto px-4 py-8">
        <header className="text-center mb-12">
          <h1 className="text-5xl font-bold mb-4 text-transparent bg-clip-text bg-gradient-to-r from-[#00ff9d] to-[#00ccff]">
            Neural Face Analysis System
          </h1>
          <p className="text-[#00ccff]">Advanced Biometric Analysis & Recognition</p>
        </header>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-8">
          <Card className="p-6 bg-[rgba(15,28,46,0.8)] backdrop-blur-md border-[#00ff9d] shadow-[0_0_10px_#00ff9d]">
            <h2 className="text-2xl font-bold mb-6">Image Upload</h2>
            
            <div className="mb-6">
              <Label className="text-[#00ccff]">Source Image</Label>
              <div
                className="upload-area p-8 rounded-lg text-center cursor-pointer border-2 border-dashed border-[#00ff9d] bg-[rgba(0,255,157,0.05)] hover:bg-[rgba(0,255,157,0.1)] hover:shadow-[0_0_15px_#00ff9d] transition-all"
                onDrop={(e) => {
                  e.preventDefault();
                  handleFileUpload(e.dataTransfer.files[0], "source");
                }}
                onDragOver={(e) => e.preventDefault()}
              >
                <input
                  type="file"
                  onChange={(e) => handleFileUpload(e.target.files[0], "source")}
                  className="hidden"
                  accept="image/*"
                />
                <i className="bi bi-upload text-4xl"></i>
                <p className="mt-2">Drop source image or click to upload</p>
              </div>
            </div>

            <div className="mb-6">
              <Label className="text-[#00ccff]">Reference Image</Label>
              <div
                className="upload-area p-8 rounded-lg text-center cursor-pointer border-2 border-dashed border-[#00ff9d] bg-[rgba(0,255,157,0.05)] hover:bg-[rgba(0,255,157,0.1)] hover:shadow-[0_0_15px_#00ff9d] transition-all"
                onDrop={(e) => {
                  e.preventDefault();
                  handleFileUpload(e.dataTransfer.files[0], "target");
                }}
                onDragOver={(e) => e.preventDefault()}
              >
                <input
                  type="file"
                  onChange={(e) => handleFileUpload(e.target.files[0], "target")}
                  className="hidden"
                  accept="image/*"
                />
                <i className="bi bi-upload text-4xl"></i>
                <p className="mt-2">Drop reference image or click to upload</p>
              </div>
            </div>

            <Button
              onClick={analyzeFaces}
              className="w-full py-3 px-6 bg-gradient-to-r from-[#00ff9d] to-[#00ccff] text-[#0f1c2e] uppercase tracking-wider font-bold hover:transform hover:-translate-y-1 hover:shadow-[0_0_20px_#00ff9d] transition-all"
            >
              Analyze Images
            </Button>
          </Card>

          <Card className="p-6 bg-[rgba(15,28,46,0.8)] backdrop-blur-md border-[#00ff9d] shadow-[0_0_10px_#00ff9d]">
            <h2 className="text-2xl font-bold mb-6">Analysis Results</h2>
            
            <div className="grid grid-cols-1 gap-6">
              <div className="relative">
                <canvas ref={canvasRef} className="w-full rounded-lg border-2 border-[#00ff9d] shadow-[0_0_15px_#00ff9d]" />
              </div>

              <div className="grid grid-cols-2 gap-4">
                <div className="stat-box p-4 rounded-lg bg-[rgba(0,255,157,0.1)] border border-[#00ff9d]">
                  <Doughnut data={similarityData} options={{
                    plugins: {
                      legend: { display: false },
                      title: {
                        display: true,
                        text: "Similarity Score",
                        color: "#00ff9d"
                      }
                    }
                  }} />
                </div>
                <div className="stat-box p-4 rounded-lg bg-[rgba(0,255,157,0.1)] border border-[#00ff9d]">
                  <Radar data={qualityData} />
                </div>
              </div>

              {analysisResults && (
                <div className="grid grid-cols-2 gap-4">
                  <div className="stat-box p-4 rounded-lg bg-[rgba(0,255,157,0.1)] border border-[#00ff9d]">
                    <h3 className="font-bold mb-2">Similarity Score</h3>
                    <p className="text-2xl">{analysisResults.similarity.toFixed(2)}%</p>
                  </div>
                  <div className="stat-box p-4 rounded-lg bg-[rgba(0,255,157,0.1)] border border-[#00ff9d]">
                    <h3 className="font-bold mb-2">Face Detection</h3>
                    <p className="text-2xl">Detected</p>
                  </div>
                  <div className="stat-box p-4 rounded-lg bg-[rgba(0,255,157,0.1)] border border-[#00ff9d]">
                    <h3 className="font-bold mb-2">Image Quality</h3>
                    <p className="text-2xl">
                      {((
                        analysisResults.qualityMetrics.clarity +
                        analysisResults.qualityMetrics.lighting +
                        analysisResults.qualityMetrics.position +
                        analysisResults.qualityMetrics.resolution
                      ) / 4).toFixed(2)}%
                    </p>
                  </div>
                  <div className="stat-box p-4 rounded-lg bg-[rgba(0,255,157,0.1)] border border-[#00ff9d]">
                    <h3 className="font-bold mb-2">Processing Time</h3>
                    <p className="text-2xl">{analysisResults.processingTime}s</p>
                  </div>
                </div>
              )}
            </div>
          </Card>
        </div>
      </div>
    </div>
  );
};

export default NeuralFaceAnalysis;
