# mdasi-sng-app
import React, { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Slider } from "@/components/ui/slider";
import { Input } from "@/components/ui/input";
import { Tabs, TabsList, TabsTrigger, TabsContent } from "@/components/ui/tabs";
import jsPDF from "jspdf";

const symptoms = [
  "疲倦感", "疼痛", "呼吸困難", "噁心", "食慾不振",
  "嗜睡", "憂鬱", "記憶困難", "打噴嚏、咳嗽或喉嚨痛",
  "影響日常生活的痛", "麻木或刺痛感", "情緒困擾", "其他症狀"
];

const interference = [
  "一般活動", "情緒", "走路能力", "工作（包括家務）", "人際關係", "生活享受"
];

export default function App() {
  const [tab, setTab] = useState("mdasi");
  const [mdasiScores, setMdasiScores] = useState(Array(19).fill(0));
  const [info, setInfo] = useState({ name: "", id: "", date: "" });
  const [painData, setPainData] = useState({
    nowPain: "", nowSoreness: "", nowNumb: "",
    maxPain: "", maxSoreness: "", maxNumb: "",
    morphine: "", painDuration: "", sorenessDuration: ""
  });

  const handleSliderChange = (index, value) => {
    const newScores = [...mdasiScores];
    newScores[index] = value;
    setMdasiScores(newScores);
  };

  const exportPDF = () => {
    const doc = new jsPDF();
    doc.setFontSize(14);
    doc.text("MDASI + Sng Pain Mapping 簡易報告", 10, 10);
    doc.text(`姓名: ${info.name}  病歷號: ${info.id}  日期: ${info.date}`, 10, 20);
    symptoms.forEach((symptom, i) => {
      doc.text(`${symptom}: ${mdasiScores[i]}`, 10, 30 + i * 7);
    });
    interference.forEach((item, i) => {
      doc.text(`${item}: ${mdasiScores[i + symptoms.length]}`, 110, 30 + i * 7);
    });
    let y = 30 + symptoms.length * 7;
    doc.text("\n【Sng Pain Mapping】", 10, y);
    y += 10;
    Object.entries(painData).forEach(([key, val]) => {
      doc.text(`${key}: ${val}`, 10, y);
      y += 7;
    });
    doc.save("mdasi_sng_report.pdf");
  };

  return (
    <div className="p-4 bg-white min-h-screen text-gray-800">
      <h1 className="text-2xl font-semibold mb-4">症狀評估工具</h1>
      <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
        <Input placeholder="姓名" value={info.name} onChange={(e) => setInfo({ ...info, name: e.target.value })} />
        <Input placeholder="病歷號" value={info.id} onChange={(e) => setInfo({ ...info, id: e.target.value })} />
        <Input placeholder="日期" value={info.date} onChange={(e) => setInfo({ ...info, date: e.target.value })} />
      </div>
      <Tabs value={tab} onValueChange={setTab}>
        <TabsList className="mb-4">
          <TabsTrigger value="mdasi">MDASI</TabsTrigger>
          <TabsTrigger value="sng">Sng Pain Mapping</TabsTrigger>
        </TabsList>

        <TabsContent value="mdasi">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            {symptoms.map((item, index) => (
              <Card key={index}>
                <CardContent>
                  <label className="block mb-2 font-medium">{item}</label>
                  <Slider
                    defaultValue={[mdasiScores[index]]}
                    max={10}
                    step={1}
                    onValueChange={(val) => handleSliderChange(index, val[0])}
                  />
                  <p className="text-right mt-1">{mdasiScores[index]}</p>
                </CardContent>
              </Card>
            ))}
            {interference.map((item, index) => (
              <Card key={index + symptoms.length}>
                <CardContent>
                  <label className="block mb-2 font-medium">{item}</label>
                  <Slider
                    defaultValue={[mdasiScores[index + symptoms.length]]}
                    max={10}
                    step={1}
                    onValueChange={(val) => handleSliderChange(index + symptoms.length, val[0])}
                  />
                  <p className="text-right mt-1">{mdasiScores[index + symptoms.length]}</p>
                </CardContent>
              </Card>
            ))}
          </div>
        </TabsContent>

        <TabsContent value="sng">
          <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
            <Input placeholder="現在痛" value={painData.nowPain} onChange={(e) => setPainData({ ...painData, nowPain: e.target.value })} />
            <Input placeholder="現在痠" value={painData.nowSoreness} onChange={(e) => setPainData({ ...painData, nowSoreness: e.target.value })} />
            <Input placeholder="現在麻" value={painData.nowNumb} onChange={(e) => setPainData({ ...painData, nowNumb: e.target.value })} />
            <Input placeholder="最痛" value={painData.maxPain} onChange={(e) => setPainData({ ...painData, maxPain: e.target.value })} />
            <Input placeholder="最痠" value={painData.maxSoreness} onChange={(e) => setPainData({ ...painData, maxSoreness: e.target.value })} />
            <Input placeholder="最麻" value={painData.maxNumb} onChange={(e) => setPainData({ ...painData, maxNumb: e.target.value })} />
            <Input placeholder="嗎啡藥物緩解 (1=無, 2=有)" value={painData.morphine} onChange={(e) => setPainData({ ...painData, morphine: e.target.value })} />
            <Input placeholder="痛多久 (1=天, 2=週, 3=月, 4=年)" value={painData.painDuration} onChange={(e) => setPainData({ ...painData, painDuration: e.target.value })} />
            <Input placeholder="痠多久 (1=天, 2=週, 3=月, 4=年)" value={painData.sorenessDuration} onChange={(e) => setPainData({ ...painData, sorenessDuration: e.target.value })} />
          </div>
        </TabsContent>
      </Tabs>
      <div className="mt-6 text-center">
        <Button onClick={exportPDF}>匯出 PDF</Button>
      </div>
    </div>
  );
}
