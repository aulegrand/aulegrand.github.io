import { useState, useRef } from "react";

const TYPES = ["DIA", "DP", "PC", "CU", "Autre"];
const AVIS = ["Favorable", "Défavorable", "Favorable avec réserves", "Sursis à statuer"];
const AVIS_COLORS = {
  "Favorable":                  { bg: "#eaf3de", color: "#3b6d11", border: "#639922" },
  "Défavorable":                { bg: "#fcebeb", color: "#a32d2d", border: "#e24b4a" },
  "Favorable avec réserves":    { bg: "#faeeda", color: "#854f0b", border: "#ba7517" },
  "Sursis à statuer":           { bg: "#eeedfe", color: "#3c3489", border: "#7f77dd" },
};

const today = () => new Date().toLocaleDateString("fr-FR");
let _nextId = 3;

function emptyDossier(id) {
  return { id, ref: "", type: "", petitionnaire: "", avis: "", obs: [{ date: today(), texte: "" }] };
}

const S = {
  app: { maxWidth: 780, margin: "0 auto", padding: "24px 16px 80px", fontFamily: "var(--font-sans)" },
  topbar: {
    background: "#1a3a5c", padding: "14px 20px", display: "flex",
    alignItems: "center", justifyContent: "space-between", gap: 12, marginBottom: 24,
    borderRadius: "var(--border-radius-lg)",
  },
  topbarTitle: { color: "#fff", fontSize: 15, fontWeight: 500, letterSpacing: "0.04em" },
  topbarSub: { color: "rgba(255,255,255,0.55)", fontSize: 11, fontWeight: 400, marginTop: 2 },
  card: {
    background: "var(--color-background-primary)",
    border: "0.5px solid var(--color-border-tertiary)",
    borderRadius: "var(--border-radius-lg)",
    marginBottom: 16, overflow: "hidden",
  },
  cardHeader: {
    background: "#1a3a5c", color: "#fff", fontSize: 11,
    fontWeight: 500, letterSpacing: "0.09em", textTransform: "uppercase",
    padding: "9px 16px", display: "flex", alignItems: "center", justifyContent: "space-between",
  },
  fieldRow: {
    display: "grid", gridTemplateColumns: "130px 1fr",
    borderBottom: "0.5px solid var(--color-border-tertiary)",
  },
  fieldLabel: {
    fontSize: 11, fontWeight: 500, color: "var(--color-text-secondary)",
    letterSpacing: "0.07em", textTransform: "uppercase",
    padding: "10px 12px 10px 16px",
    background: "var(--color-background-secondary)",
    borderRight: "0.5px solid var(--color-border-tertiary)",
    display: "flex", alignItems: "center",
  },
  fieldInput: { padding: "4px 0", display: "flex", alignItems: "center" },
  input: {
    width: "100%", border: "none", background: "transparent",
    fontFamily: "var(--font-sans)", fontSize: 14, color: "var(--color-text-primary)",
    outline: "none", padding: "6px 12px",
  },
  textarea: {
    width: "100%", border: "none", background: "transparent",
    fontFamily: "var(--font-sans)", fontSize: 14, color: "var(--color-text-primary)",
    outline: "none", padding: "8px 12px", resize: "none", lineHeight: 1.5,
  },
  pill: (active) => ({
    border: `1.5px solid ${active ? "#1a3a5c" : "var(--color-border-secondary)"}`,
    background: active ? "#1a3a5c" : "transparent",
    color: active ? "#fff" : "var(--color-text-secondary)",
    fontSize: 12, fontWeight: 500, letterSpacing: "0.06em",
    padding: "5px 13px", cursor: "pointer", fontFamily: "var(--font-sans)",
    borderRadius: "var(--border-radius-md)", transition: "all 0.12s",
  }),
  avisBtn: (active, avis) => {
    const c = AVIS_COLORS[avis];
    return {
      border: `1.5px solid ${active ? c.border : "var(--color-border-secondary)"}`,
      background: active ? c.bg : "transparent",
      color: active ? c.color : "var(--color-text-secondary)",
      fontSize: 12, fontWeight: 500, padding: "5px 12px",
      cursor: "pointer", fontFamily: "var(--font-sans)",
      borderRadius: "var(--border-radius-md)", transition: "all 0.12s",
    };
  },
  btnPrimary: {
    background: "#1a3a5c", color: "#fff", border: "none",
    fontFamily: "var(--font-sans)", fontSize: 13, fontWeight: 500,
    letterSpacing: "0.06em", textTransform: "uppercase",
    padding: "11px 24px", cursor: "pointer",
    borderRadius: "var(--border-radius-md)", flex: 1,
  },
  btnOutline: {
    background: "transparent", color: "#1a3a5c",
    border: "1.5px solid #1a3a5c",
    fontFamily: "var(--font-sans)", fontSize: 13, fontWeight: 500,
    letterSpacing: "0.06em", textTransform: "uppercase",
    padding: "9px 24px", cursor: "pointer",
    borderRadius: "var(--border-radius-md)", flex: 1,
  },
  btnSmall: {
    background: "none", border: "none",
    fontFamily: "var(--font-sans)", fontSize: 11, fontWeight: 500,
    letterSpacing: "0.06em", textTransform: "uppercase",
    color: "#1a3a5c", cursor: "pointer", padding: "2px 0",
    textDecoration: "underline", textUnderlineOffset: 3,
  },
  btnDanger: {
    background: "none", border: "none",
    fontFamily: "var(--font-sans)", fontSize: 11, fontWeight: 500,
    letterSpacing: "0.06em", textTransform: "uppercase",
    color: "#a32d2d", cursor: "pointer", padding: "2px 0",
    textDecoration: "underline", textUnderlineOffset: 3,
  },
};

function DossierCard({ d, idx, total, update, remove }) {
  const set = (k, v) => update({ ...d, [k]: v });
  const setObs = (i, k, v) => update({ ...d, obs: d.obs.map((o, j) => j === i ? { ...o, [k]: v } : o) });
  const addObs = () => update({ ...d, obs: [...d.obs, { date: today(), texte: "" }] });
  const delObs = (i) => update({ ...d, obs: d.obs.filter((_, j) => j !== i) });

  return (
    <div style={S.card}>
      <div style={S.cardHeader}>
        <span>Dossier {idx + 1}{d.ref ? ` — ${d.ref}` : ""}</span>
        {total > 1 && (
          <button onClick={remove} style={{ background: "none", border: "none", color: "rgba(255,255,255,0.45)", fontSize: 20, cursor: "pointer", lineHeight: 1, padding: 0 }}>×</button>
        )}
      </div>
      <div>
        {[
          ["Référence", <input style={S.input} placeholder="Ex. PC 001 2026…" value={d.ref} onChange={e => set("ref", e.target.value)} />],
          ["Type", (
            <div style={{ display: "flex", gap: 6, flexWrap: "wrap", padding: "8px 12px" }}>
              {TYPES.map(t => <button key={t} style={S.pill(d.type === t)} onClick={() => set("type", d.type === t ? "" : t)}>{t}</button>)}
            </div>
          )],
          ["Pétitionnaire", <input style={S.input} placeholder="Nom / organisme…" value={d.petitionnaire} onChange={e => set("petitionnaire", e.target.value)} />],
          ["Avis", (
            <div style={{ display: "flex", gap: 6, flexWrap: "wrap", padding: "8px 12px" }}>
              {AVIS.map(a => <button key={a} style={S.avisBtn(d.avis === a, a)} onClick={() => set("avis", d.avis === a ? "" : a)}>{a}</button>)}
            </div>
          )],
        ].map(([label, input], i) => (
          <div key={i} style={S.fieldRow}>
            <div style={S.fieldLabel}>{label}</div>
            <div style={{ ...S.fieldInput, alignItems: "flex-start" }}>{input}</div>
          </div>
        ))}

        <div style={{ ...S.fieldRow, borderBottom: "none", gridTemplateColumns: "130px 1fr" }}>
          <div style={{ ...S.fieldLabel, alignSelf: "flex-start", paddingTop: 14 }}>Observations</div>
          <div style={{ padding: "10px 12px" }}>
            {d.obs.map((o, i) => (
              <div key={i} style={{ display: "grid", gridTemplateColumns: "108px 1fr", gap: 8, marginBottom: 10, borderLeft: "2.5px solid var(--color-border-secondary)", paddingLeft: 10 }}>
                <input style={{ ...S.input, fontSize: 12, fontWeight: 500, color: "#1a3a5c", padding: "5px 6px" }}
                  value={o.date} placeholder="jj/mm/aaaa" onChange={e => setObs(i, "date", e.target.value)} />
                <div>
                  <textarea style={{ ...S.textarea, minHeight: 52 }} rows={2}
                    placeholder="Observations…" value={o.texte} onChange={e => setObs(i, "texte", e.target.value)} />
                  {d.obs.length > 1 && (
                    <button style={S.btnDanger} onClick={() => delObs(i)}>Supprimer</button>
                  )}
                </div>
              </div>
            ))}
            <button style={S.btnSmall} onClick={addObs}>+ Observation</button>
          </div>
        </div>
      </div>
    </div>
  );
}

export default function App() {
  const [num, setNum] = useState("");
  const [date, setDate] = useState(today());
  const [lieu, setLieu] = useState("");
  const [presents, setPresents] = useState("");
  const [excuses, setExcuses] = useState("");
  const [divers, setDivers] = useState("");
  const [dossiers, setDossiers] = useState([emptyDossier(1), emptyDossier(2)]);
  const [exporting, setExporting] = useState(false);

  const updDossier = (i, d) => setDossiers(ds => ds.map((x, j) => j === i ? d : x));
  const addDossier = () => { const id = _nextId++; setDossiers(ds => [...ds, emptyDossier(id)]); };
  const delDossier = (i) => setDossiers(ds => ds.filter((_, j) => j !== i));

  const reset = () => {
    if (!window.confirm("Réinitialiser le formulaire ?")) return;
    setNum(""); setDate(today()); setLieu("");
    setPresents(""); setExcuses(""); setDivers("");
    _nextId = 3;
    setDossiers([emptyDossier(1), emptyDossier(2)]);
  };

  const exportPDF = async () => {
    setExporting(true);
    try {
      const { jsPDF } = window.jspdf;
      const doc = new jsPDF({ orientation: "portrait", unit: "mm", format: "a4" });
      const W = 210, M = 18, CW = W - M * 2;
      let y = 0;

      const BLUE = [26, 58, 92];
      const WHITE = [255, 255, 255];
      const LGRAY = [245, 243, 239];
      const GRAY = [100, 100, 100];
      const BORDER = [210, 205, 198];

      const checkY = (need) => {
        if (y + need > 272) { doc.addPage(); y = 18; }
      };

      // Header band
      doc.setFillColor(...BLUE);
      doc.rect(0, 0, 210, 22, "F");
      doc.setFont("helvetica", "bold");
      doc.setFontSize(13);
      doc.setTextColor(...WHITE);
      doc.text("COMMISSION URBANISME", M, 10);
      doc.setFont("helvetica", "normal");
      doc.setFontSize(8);
      doc.setTextColor(180, 200, 220);
      doc.text("Compte rendu de séance", M, 16);
      if (num) {
        doc.setFont("helvetica", "bold");
        doc.setFontSize(9);
        doc.setTextColor(...WHITE);
        doc.text(`N° ${num}`, W - M, 13, { align: "right" });
      }
      y = 30;

      // Meta line
      doc.setFont("helvetica", "normal");
      doc.setFontSize(8.5);
      doc.setTextColor(...GRAY);
      const meta = [date, lieu].filter(Boolean).join("  ·  ");
      if (meta) { doc.text(meta, M, y); y += 6; }

      // Separator
      doc.setDrawColor(...BORDER);
      doc.setLineWidth(0.3);
      doc.line(M, y, W - M, y);
      y += 8;

      const sectionTitle = (title) => {
        checkY(10);
        doc.setFont("helvetica", "bold");
        doc.setFontSize(8);
        doc.setTextColor(...BLUE);
        doc.text(title.toUpperCase(), M, y);
        doc.setDrawColor(...BLUE);
        doc.setLineWidth(0.4);
        doc.line(M, y + 1.5, W - M, y + 1.5);
        y += 8;
      };

      const tableRow = (label, value, shade = false) => {
        const lw = 38, rw = CW - lw;
        const lines = doc.splitTextToSize(value || "—", rw - 4);
        const h = Math.max(7, lines.length * 5 + 3);
        checkY(h);
        if (shade) { doc.setFillColor(...LGRAY); doc.rect(M, y - 4, lw, h, "F"); }
        doc.setFillColor(255, 255, 255);
        doc.rect(M + lw, y - 4, rw, h, "F");
        doc.setDrawColor(...BORDER);
        doc.setLineWidth(0.2);
        doc.rect(M, y - 4, CW, h, "S");
        doc.line(M + lw, y - 4, M + lw, y - 4 + h);
        doc.setFont("helvetica", "bold");
        doc.setFontSize(7.5);
        doc.setTextColor(...GRAY);
        doc.text(label.toUpperCase(), M + 2, y);
        doc.setFont("helvetica", "normal");
        doc.setFontSize(8.5);
        doc.setTextColor(...[40, 35, 30]);
        doc.text(lines, M + lw + 3, y);
        y += h;
      };

      // Présence
      sectionTitle("Présence");
      tableRow("Présents", presents, true);
      tableRow("Excusés", excuses);
      y += 6;

      // Dossiers
      sectionTitle("Dossiers examinés");
      dossiers.forEach((d, i) => {
        checkY(12);
        // Dossier header
        doc.setFillColor(...BLUE);
        doc.rect(M, y - 4, CW, 10, "F");
        doc.setFont("helvetica", "bold");
        doc.setFontSize(8.5);
        doc.setTextColor(...WHITE);
        doc.text(`Dossier ${i + 1}${d.ref ? " — " + d.ref : ""}`, M + 3, y + 1);
        y += 10;

        tableRow("Type", d.type, true);
        tableRow("Pétitionnaire", d.petitionnaire);
        tableRow("Avis", d.avis);

        // Observations
        const filled = d.obs.filter(o => o.texte.trim());
        if (filled.length > 0) {
          const obsText = filled.map(o => `[${o.date}]  ${o.texte}`).join("\n");
          const lines = doc.splitTextToSize(obsText, CW - 44);
          const h = Math.max(7, lines.length * 5 + 3);
          checkY(h);
          doc.setFillColor(...LGRAY);
          doc.rect(M, y - 4, 38, h, "F");
          doc.setFillColor(255, 255, 255);
          doc.rect(M + 38, y - 4, CW - 38, h, "F");
          doc.setDrawColor(...BORDER);
          doc.setLineWidth(0.2);
          doc.rect(M, y - 4, CW, h, "S");
          doc.line(M + 38, y - 4, M + 38, y - 4 + h);
          doc.setFont("helvetica", "bold");
          doc.setFontSize(7.5);
          doc.setTextColor(...GRAY);
          doc.text("OBSERVATIONS", M + 2, y);
          doc.setFont("helvetica", "normal");
          doc.setFontSize(8);
          doc.setTextColor(40, 35, 30);
          doc.text(lines, M + 41, y);
          y += h;
        } else {
          tableRow("Observations", "—");
        }
        y += 5;
      });

      // Points divers
      if (divers.trim()) {
        y += 2;
        sectionTitle("Points divers");
        const lines = doc.splitTextToSize(divers, CW);
        checkY(lines.length * 5 + 4);
        doc.setFont("helvetica", "normal");
        doc.setFontSize(8.5);
        doc.setTextColor(40, 35, 30);
        doc.text(lines, M, y);
        y += lines.length * 5 + 4;
      }

      // Footer on each page
      const pages = doc.getNumberOfPages();
      for (let p = 1; p <= pages; p++) {
        doc.setPage(p);
        doc.setDrawColor(...BORDER);
        doc.setLineWidth(0.3);
        doc.line(M, 285, W - M, 285);
        doc.setFont("helvetica", "normal");
        doc.setFontSize(7);
        doc.setTextColor(160, 150, 140);
        doc.text("Document confidentiel — réservé aux membres de la commission", M, 290);
        doc.text(`Page ${p} / ${pages}`, W - M, 290, { align: "right" });
      }

      const filename = `CR_Commission_Urbanisme${num ? "_N" + num : ""}_${date.replace(/\//g, "-")}.pdf`;
      doc.save(filename);
    } catch (e) {
      alert("Erreur lors de la génération du PDF : " + e.message);
    }
    setExporting(false);
  };

  return (
    <>
      <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js" />
      <div style={S.app}>
        <h2 className="sr-only">Formulaire de compte rendu — Commission Urbanisme</h2>

        {/* Topbar */}
        <div style={S.topbar}>
          <div>
            <div style={S.topbarTitle}>Commission Urbanisme</div>
            <div style={S.topbarSub}>Compte rendu de séance</div>
          </div>
          <div style={{ display: "flex", alignItems: "center", gap: 8, color: "rgba(255,255,255,0.7)", fontSize: 11 }}>
            <span style={{ letterSpacing: "0.06em", textTransform: "uppercase" }}>N°</span>
            <input value={num} onChange={e => setNum(e.target.value)}
              placeholder="—" maxLength={8}
              style={{ background: "rgba(255,255,255,0.12)", border: "1px solid rgba(255,255,255,0.25)", color: "#fff", fontFamily: "var(--font-sans)", fontSize: 15, fontWeight: 500, width: 52, textAlign: "center", padding: "4px 6px", borderRadius: "var(--border-radius-md)", outline: "none" }} />
          </div>
        </div>

        {/* Infos générales */}
        <div style={S.card}>
          <div style={S.cardHeader}>Informations générales</div>
          {[
            ["Date", <input style={S.input} value={date} onChange={e => setDate(e.target.value)} placeholder="jj/mm/aaaa" />],
            ["Lieu", <input style={S.input} value={lieu} onChange={e => setLieu(e.target.value)} placeholder="Salle du conseil…" />],
            ["Présents", <textarea style={{ ...S.textarea, minHeight: 44 }} rows={2} value={presents} onChange={e => setPresents(e.target.value)} placeholder="Noms des membres présents…" />],
            ["Excusés", <textarea style={{ ...S.textarea, minHeight: 36 }} rows={2} value={excuses} onChange={e => setExcuses(e.target.value)} placeholder="Noms des membres excusés…" />],
          ].map(([label, input], i, arr) => (
            <div key={i} style={{ ...S.fieldRow, borderBottom: i < arr.length - 1 ? "0.5px solid var(--color-border-tertiary)" : "none" }}>
              <div style={S.fieldLabel}>{label}</div>
              <div style={{ ...S.fieldInput, alignItems: "flex-start" }}>{input}</div>
            </div>
          ))}
        </div>

        {/* Dossiers */}
        {dossiers.map((d, i) => (
          <DossierCard key={d.id} d={d} idx={i} total={dossiers.length}
            update={nd => updDossier(i, nd)} remove={() => delDossier(i)} />
        ))}

        <button onClick={addDossier} style={{
          width: "100%", padding: 13,
          border: "1.5px dashed var(--color-border-secondary)", background: "transparent",
          fontFamily: "var(--font-sans)", fontSize: 12, fontWeight: 500, letterSpacing: "0.07em",
          textTransform: "uppercase", color: "var(--color-text-secondary)", cursor: "pointer",
          borderRadius: "var(--border-radius-lg)", marginBottom: 16,
        }}>+ Ajouter un dossier</button>

        {/* Points divers */}
        <div style={S.card}>
          <div style={S.cardHeader}>Points divers</div>
          <textarea style={{ ...S.textarea, width: "100%", minHeight: 72, padding: "10px 16px" }}
            rows={3} value={divers} onChange={e => setDivers(e.target.value)}
            placeholder="Notes, questions diverses, annonces…" />
        </div>

        {/* Actions */}
        <div style={{ display: "flex", gap: 12, flexWrap: "wrap", marginTop: 8 }}>
          <button style={{ ...S.btnPrimary, opacity: exporting ? 0.7 : 1 }}
            onClick={exportPDF} disabled={exporting}>
            {exporting ? "Génération…" : "Exporter en PDF"}
          </button>
          <button style={S.btnOutline} onClick={reset}>Nouvelle séance</button>
        </div>
      </div>
    </>
  );
}
