import React, { useEffect, useState, useRef } from "react";
import { Button } from "@/components/ui/button";
import { motion } from "framer-motion";

// Default audio queues
const pauseFiles = [
  "/audio/pause1.mp3",
  "/audio/pause2.mp3",
  "/audio/pause3.mp3",
  "/audio/pause4.mp3",
  "/audio/pause5.mp3",
];

const whyFiles = [
  "/audio/why1.mp3",
  "/audio/why2.mp3",
  "/audio/why3.mp3",
  "/audio/why4.mp3",
  "/audio/why5.mp3",
];

export default function PauseApp() {
  const [queue, setQueue] = useState("pause");
  const [customAudio, setCustomAudio] = useState<string | null>(null);
  const [showSettings, setShowSettings] = useState(false);
  const [buttonText, setButtonText] = useState("PAUSE!");
  const [buttonColor, setButtonColor] = useState("#dc2626"); // default red-600
  const [backgroundColor, setBackgroundColor] = useState("#f3f4f6"); // default gray-100

  // Recorder state
  const [recording, setRecording] = useState(false);
  const mediaRecorderRef = useRef<MediaRecorder | null>(null);
  const [audioChunks, setAudioChunks] = useState<Blob[]>([]);

  // Load saved settings on mount
  useEffect(() => {
    const savedText = localStorage.getItem("buttonText");
    const savedButtonColor = localStorage.getItem("buttonColor");
    const savedBackgroundColor = localStorage.getItem("backgroundColor");
    const savedCustomAudio = localStorage.getItem("customAudio");

    if (savedText) setButtonText(savedText);
    if (savedButtonColor) setButtonColor(savedButtonColor);
    if (savedBackgroundColor) setBackgroundColor(savedBackgroundColor);
    if (savedCustomAudio) setCustomAudio(savedCustomAudio);
  }, []);

  useEffect(() => {
    const files = getCurrentFiles();
    if (files.length > 0) {
      const intro = new Audio(files[0]);
      intro.play();
    }
  }, [queue, customAudio]);

  const getCurrentFiles = () => {
    if (customAudio) return [customAudio];
    return queue === "pause" ? pauseFiles : whyFiles;
  };

  const playRandomAudio = () => {
    const files = getCurrentFiles();
    const randomIndex = Math.floor(Math.random() * files.length);
    const audio = new Audio(files[randomIndex]);
    audio.play();
  };

  const handleFileUpload = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (file) {
      const url = URL.createObjectURL(file);
      const audio = new Audio(url);

      audio.onloadedmetadata = () => {
        if (audio.duration <= 5) {
          setCustomAudio(url);
          localStorage.setItem("customAudio", url);
        } else {
          alert("File must be 5 seconds or shorter.");
        }
      };
    }
  };

  const startRecording = async () => {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    const mediaRecorder = new MediaRecorder(stream);
    mediaRecorderRef.current = mediaRecorder;
    setAudioChunks([]);
    mediaRecorder.start();
    setRecording(true);

    mediaRecorder.ondataavailable = (event) => {
      if (event.data.size > 0) {
        setAudioChunks((prev) => [...prev, event.data]);
      }
    };

    mediaRecorder.onstop = () => {
      const blob = new Blob(audioChunks, { type: "audio/mp3" });
      const url = URL.createObjectURL(blob);
      const audio = new Audio(url);
      audio.onloadedmetadata = () => {
        if (audio.duration <= 5) {
          setCustomAudio(url);
          localStorage.setItem("customAudio", url);
        } else {
          alert("Recording must be 5 seconds or shorter.");
        }
      };
    };
  };

  const stopRecording = () => {
    mediaRecorderRef.current?.stop();
    setRecording(false);
  };

  const handleTextChange = (text: string) => {
    setButtonText(text);
    localStorage.setItem("buttonText", text);
  };

  const handleButtonColorChange = (color: string) => {
    setButtonColor(color);
    localStorage.setItem("buttonColor", color);
  };

  const handleBackgroundColorChange = (color: string) => {
    setBackgroundColor(color);
    localStorage.setItem("backgroundColor", color);
  };

  const resetSettings = () => {
    setButtonText("PAUSE!");
    setButtonColor("#dc2626");
    setBackgroundColor("#f3f4f6");
    setCustomAudio(null);
    setQueue("pause");
    localStorage.clear();
  };

  return (
    <div className={`flex flex-col items-center justify-center h-screen gap-6`} style={{ backgroundColor }}>
      {showSettings ? (
        <div className="flex flex-col gap-4 p-6 bg-white rounded-2xl shadow-lg w-80">
          <h2 className="text-xl font-bold">Settings</h2>

          {/* Upload */}
          <label className="flex flex-col gap-2">
            <span className="text-sm">Upload custom MP3/MP4 (max 5s)</span>
            <input type="file" accept="audio/mp3, audio/mp4" onChange={handleFileUpload} />
          </label>

          {/* Recorder */}
          <div className="flex gap-2 items-center">
            {recording ? (
              <Button className="bg-red-500 text-white" onClick={stopRecording}>
                Stop Recording
              </Button>
            ) : (
              <Button className="bg-green-500 text-white" onClick={startRecording}>
                Start Recording
              </Button>
            )}
          </div>

          {/* Button Text */}
          <label className="flex flex-col gap-2">
            <span className="text-sm">Button Text (max 20 chars)</span>
            <input
              type="text"
              maxLength={20}
              value={buttonText}
              onChange={(e) => handleTextChange(e.target.value)}
              className="border px-2 py-1 rounded"
            />
          </label>

          {/* Colors */}
          <label className="flex flex-col gap-2">
            <span className="text-sm">Button Color</span>
            <input type="color" value={buttonColor} onChange={(e) => handleButtonColorChange(e.target.value)} />
          </label>

          <label className="flex flex-col gap-2">
            <span className="text-sm">Background Color</span>
            <input type="color" value={backgroundColor} onChange={(e) => handleBackgroundColorChange(e.target.value)} />
          </label>

          {/* Reset */}
          <Button onClick={resetSettings} className="bg-yellow-500 text-white">
            Reset to Default
          </Button>

          <Button onClick={() => setShowSettings(false)} className="bg-blue-500 text-white">
            Back
          </Button>
        </div>
      ) : (
        <>
          <motion.div whileTap={{ scale: 0.9 }} whileHover={{ scale: 1.1 }}>
            <Button
              className={`w-56 h-56 rounded-full text-4xl font-extrabold shadow-xl text-white`}
              style={{ backgroundColor: buttonColor }}
              onClick={playRandomAudio}
            >
              {buttonText}
            </Button>
          </motion.div>

          <div className="flex gap-4">
            <Button
              className={`px-4 py-2 rounded-lg font-bold shadow ${
                queue === "pause" && !customAudio ? "bg-red-500 text-white" : "bg-gray-200"
              }`}
              onClick={() => {
                setCustomAudio(null);
                setQueue("pause");
                handleTextChange("PAUSE!");
              }}
            >
              Pause Queue
            </Button>

            <Button
              className={`px-4 py-2 rounded-lg font-bold shadow ${
                queue === "why" && !customAudio ? "bg-red-500 text-white" : "bg-gray-200"
              }`}
              onClick={() => {
                setCustomAudio(null);
                setQueue("why");
                handleTextChange("WHY!");
              }}
            >
              Why Queue
            </Button>

            <Button className="px-4 py-2 rounded-lg font-bold shadow bg-blue-500 text-white" onClick={() => setShowSettings(true)}>
              Settings
            </Button>
          </div>
        </>
      )}
    </div>
  );
}
