# import React, { useState } from 'react';

// Advanced React demo for "Legal Simplifier" hackathon MVP.
// Features: live clause highlighting, sentiment-like risk coloring, multiple export options, and API stubs.
// Tailwind CSS classes are used.

export default function LegalSimplifierDemo() {
  const [fileText, setFileText] = useState('');
  const [inputText, setInputText] = useState('');
  const [docType, setDocType] = useState('unspecified');
  const [jurisdiction, setJurisdiction] = useState('IN');
  const [loading, setLoading] = useState(false);
  const [result, setResult] = useState(null);
  const [error, setError] = useState(null);

  async function handleSimplify() {
    const textToSend = inputText || fileText;
    if (!textToSend.trim()) {
      setError('Please paste or upload a document text to simplify.');
      return;
    }
    setError(null);
    setLoading(true);
    setResult(null);
    try {
      const resp = await fetch('/api/simplify', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ text: textToSend, docType, jurisdiction }),
      });
      if (!resp.ok) throw new Error(`API error: ${resp.status}`);
      const data = await resp.json();
      setResult(data);
    } catch (e) {
      setError(String(e));
    } finally {
      setLoading(false);
    }
  }

  function handleFile(e) {
    const f = e.target.files && e.target.files[0];
    if (!f) return;
    const reader = new FileReader();
    reader.onload = (ev) => {
      setFileText(String(ev.target.result || ''));
    };
    reader.readAsText(f);
  }

  function riskColor(level) {
    switch (level) {
      case 'High': return 'text-red-600';
      case 'Medium': return 'text-yellow-600';
      case 'Low': return 'text-green-600';
      default: return 'text-gray-700';
    }
  }

  return (
    <div className="max-w-5xl mx-auto p-6">
      <header className="mb-6">
        <h1 className="text-3xl font-bold">⚖️ Legal Simplifier — Advanced Demo</h1>
        <p className="text-sm text-gray-600 mt-2">Transform complex legal jargon into clear, actionable guidance with risk insights, provenance, and export tools.</p>
      </header>

      <section className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div className="space-y-4">
          <label className="block">
            <span className="text-sm font-medium">Upload file (plain-text for demo)</span>
            <input type="file" onChange={handleFile} className="mt-2 block w-full text-sm text-gray-700" />
          </label>

          <label className="block">
            <span className="text-sm font-medium">Or paste clause / text</span>
            <textarea value={inputText} onChange={(e) => setInputText(e.target.value)} rows={8} className="mt-2 block w-full rounded-lg border p-2"></textarea>
          </label>

          <div className="flex gap-3">
            <label className="flex-1">
              <span className="text-sm font-medium">Document type</span>
              <select value={docType} onChange={(e) => setDocType(e.target.value)} className="mt-2 block w-full rounded border p-2">
                <option value="unspecified">Auto-detect</option>
                <option value="NDA">NDA</option>
                <option value="Lease">Lease / Rental</option>
                <option value="Employment">Employment</option>
                <option value="TOS">Terms of Service</option>
              </select>
            </label>

            <label>
              <span className="text-sm font-medium">Jurisdiction</span>
              <input value={jurisdiction} onChange={(e) => setJurisdiction(e.target.value)} className="mt-2 block w-32 rounded border p-2" />
            </label>
          </div>

          <div className="flex gap-3 mt-2">
            <button onClick={handleSimplify} className="px-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-700" disabled={loading}>
              {loading ? 'Simplifying...' : 'Simplify'}
            </button>
            <button onClick={() => { setInputText(''); setFileText(''); setResult(null); setError(null); }} className="px-4 py-2 border rounded">Reset</button>
          </div>

          {error && <div className="text-red-600 mt-2">{error}</div>}
        </div>

        <div>
          <div className="bg-gray-50 p-4 rounded-lg h-full">
            <h2 className="font-semibold">Preview / Result</h2>

            <div className="mt-3">
              <h3 className="text-sm font-medium">Original text (preview)</h3>
              <div className="mt-2 p-3 bg-white border rounded h-40 overflow-auto text-sm whitespace-pre-wrap">{inputText || fileText || <span className="text-gray-400">No text yet — paste or upload to preview.</span>}</div>
            </div>

            <div className="mt-4">
              <h3 className="text-sm font-medium">Simplified output</h3>
              <div className="mt-2 p-3 bg-white border rounded h-64 overflow-auto text-sm">
                {!result && <div className="text-gray-400">No result yet</div>}
                {result && (
                  <div>
                    <div className="mb-2">
                      <strong>Summary:</strong>
                      <div className="mt-1 text-sm">{result.summary}</div>
                    </div>

                    <div className="mb-2">
                      <strong>Key points:</strong>
                      <ul className="list-disc list-inside text-sm mt-1">{(result.key_points || []).map((kp, i) => <li key={i}>{kp}</li>)}</ul>
                    </div>

                    <div className="mb-2">
                      <strong>Risk:</strong> <span className={"ml-2 font-semibold " + riskColor(result.risk_level)}>{result.risk_level}</span> — <span className="text-sm">{result.risk_reason}</span>
                    </div>

                    <div className="mb-2">
                      <strong>Confidence:</strong> <span className="ml-2">{result.confidence ?? '—'}%</span>
                    </div>

                    <div className="mb-2">
                      <strong>Next steps:</strong>
                      <ol className="list-decimal list-inside text-sm mt-1">{(result.next_steps || []).map((s, i) => <li key={i}>{s}</li>)}</ol>
                    </div>

                    {result.open_questions?.length > 0 && (
                      <div className="mb-2">
                        <strong>Open questions:</strong>
                        <ul className="list-disc list-inside text-sm mt-1">{result.open_questions.map((q, i) => <li key={i}>{q}</li>)}</ul>
                      </div>
                    )}

                    <div className="mt-3 text-xs text-gray-500">
                      <div><strong>Source span:</strong></div>
                      <pre className="whitespace-pre-wrap">{result.source_span}</pre>
                    </div>
                  </div>
                )}
              </div>
            </div>

            <div className="mt-4 flex gap-2 flex-wrap">
              <button className="px-3 py-1 border rounded text-sm" onClick={() => {
                if (!result) return; const blob = new Blob([JSON.stringify(result, null, 2)], { type: 'application/json' }); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = 'simplified-result.json'; a.click(); URL.revokeObjectURL(url);
              }}>Download JSON</button>

              <button className="px-3 py-1 border rounded text-sm" onClick={() => {
                if (!result) return; const content = `Summary:\n${result.summary}\n\nKey points:\n${(result.key_points||[]).map((k,i)=>`${i+1}. ${k}`).join('\n')}\n\nNext steps:\n${(result.next_steps||[]).map((k,i)=>`${i+1}. ${k}`).join('\n')}`; const blob = new Blob([content], { type: 'text/plain' }); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = 'simplified-checklist.txt'; a.click(); URL.revokeObjectURL(url);
              }}>Export checklist</button>

              <button className="px-3 py-1 border rounded text-sm" onClick={() => {
                if (!result) return; const html = `<!DOCTYPE html><html><body><h2>Legal Simplifier Report</h2><pre>${JSON.stringify(result, null, 2)}</pre></body></html>`; const blob = new Blob([html], { type: 'text/html' }); const url = URL.createObjectURL(blob); const a = document.createElement('a'); a.href = url; a.download = 'report.html'; a.click(); URL.revokeObjectURL(url);
              }}>Export HTML</button>
            </div>

          </div>
        </div>
      </section>

      <footer className="mt-8 text-sm text-gray-500">
        <div>Disclaimer: This demo is for educational purposes only, not legal advice.</div>
      </footer>
    </div>
  );
}
