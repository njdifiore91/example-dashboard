import { useState, useMemo } from "react";

const ORG_UNITS = {
  ITC: ["PAM", "IBS", "TMH", "MCH", "DLN", "ISZ", "SAO", "CDS", "SCN", "RKS"],
  CRD: ["CRIMS Compliance", "Portfolio Mgmt (MWB)", "Server Framework", "Order Management", "Lantern", "OMB", "Paradigm (UI)", "Analytics"],
  DMC: ["DML"],
  SSIM: ["SSIM"],
};

const ROLES = ["Engineer", "Business Analyst", "QA"];
const USE_CASES = [
  "Documentation",
  "Test Case Generation",
  "Modernization/Refactor",
  "New Feature",
  "Bug Fix",
  "Security Vulnerability",
];

// Realistic use case distributions per role
const ROLE_UC_BASE = {
  Engineer: {
    "Documentation": 8,
    "Test Case Generation": 10,
    "Modernization/Refactor": 30,
    "New Feature": 28,
    "Bug Fix": 14,
    "Security Vulnerability": 10,
  },
  "Business Analyst": {
    "Documentation": 72,
    "Test Case Generation": 3,
    "Modernization/Refactor": 2,
    "New Feature": 18,
    "Bug Fix": 3,
    "Security Vulnerability": 2,
  },
  QA: {
    "Documentation": 22,
    "Test Case Generation": 38,
    "Modernization/Refactor": 3,
    "New Feature": 5,
    "Bug Fix": 28,
    "Security Vulnerability": 4,
  },
};

// Realistic avg run sizes per role
const ROLE_RUN_SIZE_BASE = {
  Engineer: { min: 85000, max: 140000 },
  "Business Analyst": { min: 14000, max: 28000 },
  QA: { min: 10000, max: 22000 },
};

function seededRandom(seed) {
  let s = seed;
  return () => {
    s = (s * 16807 + 0) % 2147483647;
    return (s - 1) / 2147483646;
  };
}

function generateMockData() {
  const rand = seededRandom(42);
  const data = {};
  Object.entries(ORG_UNITS).forEach(([org, apps]) => {
    if (apps.length === 0) return;
    data[org] = {};
    apps.forEach((app) => {
      data[org][app] = {};
      ROLES.forEach((role) => {
        const headcount = Math.floor(rand() * 12) + 3;
        const activeUsers = Math.floor(headcount * (0.4 + rand() * 0.55));

        // Use case: start from base distribution, add small variance
        const useCaseBreakdown = {};
        let ucTotal = 0;
        USE_CASES.forEach((uc) => {
          const base = ROLE_UC_BASE[role][uc];
          const variance = Math.round((rand() - 0.5) * base * 0.4);
          useCaseBreakdown[uc] = Math.max(base + variance, 1);
          ucTotal += useCaseBreakdown[uc];
        });
        USE_CASES.forEach((uc) => {
          useCaseBreakdown[uc] = Math.round((useCaseBreakdown[uc] / ucTotal) * 100);
        });

        // Run size from role-appropriate range
        const runRange = ROLE_RUN_SIZE_BASE[role];
        const avgRunSize = Math.round(runRange.min + rand() * (runRange.max - runRange.min));

        data[org][app][role] = {
          headcount,
          activeUsers,
          adoptionRate: Math.round((activeUsers / headcount) * 100),
          useCases: useCaseBreakdown,
          acceleration: Math.round(20 + rand() * 55),
          avgRunSize,
          tasksPerWeek: Math.round(5 + rand() * 35),
          runsThisPeriod: Math.round(20 + rand() * 200),
        };
      });
    });
  });
  return data;
}

const MOCK_DATA = generateMockData();

const PRIMARY = "#001AFF";
const PRIMARY_DARK = "#0012B3";
const WHITE = "#FFFFFF";
const G50 = "#F8FAFC";
const G100 = "#F1F5F9";
const G200 = "#E2E8F0";
const G300 = "#CBD5E1";
const G400 = "#94A3B8";
const G500 = "#64748B";
const G700 = "#334155";
const G900 = "#0F172A";

const ROLE_ACCENT = {
  Engineer: { bg: "#001AFF", light: "#E8ECFF", text: "#001AFF" },
  "Business Analyst": { bg: "#D97706", light: "#FEF7ED", text: "#B45309" },
  QA: { bg: "#059669", light: "#ECFDF5", text: "#047857" },
};

const UC_SHORT = {
  Documentation: "Docs",
  "Test Case Generation": "Test Gen",
  "Modernization/Refactor": "Mod/Refactor",
  "New Feature": "New Feature",
  "Bug Fix": "Bug Fix",
  "Security Vulnerability": "Security",
};

function formatRunSize(val) {
  if (val >= 1000) return `${(val / 1000).toFixed(val >= 100000 ? 0 : 1)}k`;
  return val.toString();
}

function aggregateByRole(data, orgFilter, appFilter) {
  const result = {};
  ROLES.forEach((role) => {
    let totalAccel = 0, totalRun = 0, totalTasks = 0, totalRuns = 0;
    let totalHead = 0, totalActive = 0, count = 0;
    const ucTotals = {};
    USE_CASES.forEach((uc) => { ucTotals[uc] = 0; });

    const orgsToScan = orgFilter ? [orgFilter] : Object.keys(data);
    orgsToScan.forEach((org) => {
      if (!data[org]) return;
      const appsToScan = appFilter ? [appFilter] : Object.keys(data[org]);
      appsToScan.forEach((app) => {
        const m = data[org]?.[app]?.[role];
        if (!m) return;
        totalAccel += m.acceleration;
        totalRun += m.avgRunSize;
        totalTasks += m.tasksPerWeek;
        totalRuns += m.runsThisPeriod;
        totalHead += m.headcount;
        totalActive += m.activeUsers;
        count++;
        USE_CASES.forEach((uc) => { ucTotals[uc] += m.useCases[uc] || 0; });
      });
    });

    const ucTotal = Object.values(ucTotals).reduce((a, b) => a + b, 0);
    const ucNorm = {};
    USE_CASES.forEach((uc) => { ucNorm[uc] = ucTotal > 0 ? Math.round((ucTotals[uc] / ucTotal) * 100) : 0; });

    result[role] = {
      acceleration: count > 0 ? Math.round(totalAccel / count) : 0,
      avgRunSize: count > 0 ? Math.round(totalRun / count) : 0,
      tasksPerWeek: count > 0 ? Math.round(totalTasks / count) : 0,
      runsThisPeriod: totalRuns,
      headcount: totalHead,
      activeUsers: totalActive,
      adoptionRate: totalHead > 0 ? Math.round((totalActive / totalHead) * 100) : 0,
      useCases: ucNorm,
    };
  });
  return result;
}

function AccelerationGauge({ value, accent }) {
  const radius = 54;
  const stroke = 10;
  const circumference = Math.PI * radius;
  const progress = (value / 100) * circumference;

  return (
    <div style={{ display: "flex", flexDirection: "column", alignItems: "center", padding: "8px 0" }}>
      <svg width="128" height="76" viewBox="0 0 128 76">
        <path
          d={`M ${64 - radius} 68 A ${radius} ${radius} 0 0 1 ${64 + radius} 68`}
          fill="none" stroke={G200} strokeWidth={stroke} strokeLinecap="round"
        />
        <path
          d={`M ${64 - radius} 68 A ${radius} ${radius} 0 0 1 ${64 + radius} 68`}
          fill="none" stroke={accent} strokeWidth={stroke} strokeLinecap="round"
          strokeDasharray={`${progress} ${circumference}`}
          style={{ transition: "stroke-dasharray 0.8s ease" }}
        />
        <text x="64" y="58" textAnchor="middle" style={{ fontSize: 28, fontWeight: 800, fill: G900 }}>
          {value}%
        </text>
        <text x="64" y="72" textAnchor="middle" style={{ fontSize: 9, fontWeight: 600, fill: G500, textTransform: "uppercase", letterSpacing: "0.08em" }}>
          acceleration
        </text>
      </svg>
    </div>
  );
}

function UseCaseBars({ useCases, accent }) {
  const sorted = USE_CASES.slice().sort((a, b) => useCases[b] - useCases[a]);
  const max = Math.max(...Object.values(useCases), 1);

  return (
    <div style={{ display: "flex", flexDirection: "column", gap: 5 }}>
      {sorted.map((uc) => (
        <div key={uc} style={{ display: "flex", alignItems: "center", gap: 6 }}>
          <div style={{ width: 72, fontSize: 10, fontWeight: 500, color: G500, textAlign: "right", flexShrink: 0 }}>
            {UC_SHORT[uc]}
          </div>
          <div style={{ flex: 1, height: 14, backgroundColor: G100, borderRadius: 3, overflow: "hidden" }}>
            <div style={{
              height: "100%", borderRadius: 3, backgroundColor: accent,
              width: `${Math.max((useCases[uc] / max) * 100, 3)}%`,
              opacity: 0.7 + (useCases[uc] / max) * 0.3,
              transition: "width 0.5s ease",
            }} />
          </div>
          <div style={{ width: 30, fontSize: 10, fontWeight: 700, color: G700, textAlign: "right" }}>
            {useCases[uc]}%
          </div>
        </div>
      ))}
    </div>
  );
}

function StatRow({ label, value, unit }) {
  return (
    <div style={{ display: "flex", justifyContent: "space-between", alignItems: "baseline", padding: "6px 0", borderBottom: `1px solid ${G100}` }}>
      <span style={{ fontSize: 11, fontWeight: 500, color: G500 }}>{label}</span>
      <span style={{ fontSize: 15, fontWeight: 700, color: G900 }}>
        {value}<span style={{ fontSize: 11, fontWeight: 500, color: G400, marginLeft: 2 }}>{unit}</span>
      </span>
    </div>
  );
}

function RoleColumn({ role, data }) {
  const d = data[role];
  if (!d) return null;
  const colors = ROLE_ACCENT[role];

  return (
    <div style={{
      flex: 1, minWidth: 240, backgroundColor: WHITE, borderRadius: 10,
      border: `1px solid ${G200}`, overflow: "hidden",
      display: "flex", flexDirection: "column",
    }}>
      <div style={{
        padding: "14px 20px", backgroundColor: colors.light,
        borderBottom: `2px solid ${colors.bg}`,
        display: "flex", alignItems: "center", gap: 10,
      }}>
        <div style={{ width: 8, height: 8, borderRadius: "50%", backgroundColor: colors.bg }} />
        <div style={{ fontSize: 15, fontWeight: 700, color: colors.text }}>{role}</div>
        <div style={{
          marginLeft: "auto", fontSize: 10, fontWeight: 600, color: colors.text,
          backgroundColor: colors.bg + "18", padding: "2px 8px", borderRadius: 4,
        }}>
          {d.activeUsers}/{d.headcount} active
        </div>
      </div>

      <div style={{ padding: "8px 20px 0" }}>
        <AccelerationGauge value={d.acceleration} accent={colors.bg} />
      </div>

      <div style={{ padding: "0 20px 12px" }}>
        <StatRow label="Avg Run Size" value={formatRunSize(d.avgRunSize)} unit=" LOC" />
        <StatRow label="Runs This Period" value={d.runsThisPeriod} unit="" />
        <StatRow label="Tasks / Week" value={d.tasksPerWeek} unit="" />
        <StatRow label="Adoption Rate" value={d.adoptionRate} unit="%" />
      </div>

      <div style={{ padding: "8px 20px 16px", borderTop: `1px solid ${G100}` }}>
        <div style={{
          fontSize: 10, fontWeight: 700, color: G500, textTransform: "uppercase",
          letterSpacing: "0.08em", marginBottom: 8,
        }}>
          Use Case Breakdown
        </div>
        <UseCaseBars useCases={d.useCases} accent={colors.bg} />
      </div>
    </div>
  );
}

export default function Dashboard() {
  const [selectedOrg, setSelectedOrg] = useState(null);
  const [selectedApp, setSelectedApp] = useState(null);
  const [timeRange, setTimeRange] = useState("month");

  const activeOrgs = Object.keys(MOCK_DATA);
  const availableApps = selectedOrg
    ? (ORG_UNITS[selectedOrg] || []).filter((a) => MOCK_DATA[selectedOrg]?.[a])
    : [];

  const roleData = useMemo(
    () => aggregateByRole(MOCK_DATA, selectedOrg, selectedApp),
    [selectedOrg, selectedApp]
  );

  const orgRoleAccel = useMemo(() => {
    return activeOrgs.map((org) => {
      const d = aggregateByRole(MOCK_DATA, org, null);
      return { org, ...Object.fromEntries(ROLES.map((r) => [r, d[r].acceleration])) };
    });
  }, []);

  return (
    <div style={{
      fontFamily: "'Inter', -apple-system, sans-serif",
      backgroundColor: G50, minHeight: "100vh", color: G900,
    }}>
      {/* HEADER */}
      <div style={{
        background: `linear-gradient(135deg, ${PRIMARY} 0%, ${PRIMARY_DARK} 100%)`,
        padding: "20px 32px",
        display: "flex", alignItems: "center", justifyContent: "space-between",
      }}>
        <div>
          <div style={{ fontSize: 10, fontWeight: 600, color: "rgba(255,255,255,0.50)", textTransform: "uppercase", letterSpacing: "0.1em", marginBottom: 3 }}>
            State Street × Blitzy
          </div>
          <div style={{ fontSize: 20, fontWeight: 700, color: WHITE }}>
            Platform Acceleration Dashboard
          </div>
        </div>
        <div style={{ display: "flex", alignItems: "center", gap: 6 }}>
          {["week", "month", "quarter"].map((t) => (
            <button key={t} onClick={() => setTimeRange(t)} style={{
              padding: "5px 14px", borderRadius: 6, border: "none", fontSize: 11, fontWeight: 600,
              cursor: "pointer",
              backgroundColor: timeRange === t ? WHITE : "rgba(255,255,255,0.12)",
              color: timeRange === t ? PRIMARY : "rgba(255,255,255,0.7)",
            }}>
              {t.charAt(0).toUpperCase() + t.slice(1)}
            </button>
          ))}
        </div>
      </div>

      {/* FILTER BAR */}
      <div style={{
        padding: "10px 32px", backgroundColor: WHITE,
        borderBottom: `1px solid ${G200}`,
        display: "flex", alignItems: "center", gap: 8, flexWrap: "wrap",
      }}>
        <span style={{ fontSize: 10, fontWeight: 700, color: G400, textTransform: "uppercase", letterSpacing: "0.08em", marginRight: 4 }}>
          Org Unit
        </span>
        <button onClick={() => { setSelectedOrg(null); setSelectedApp(null); }}
          style={{
            padding: "4px 12px", borderRadius: 5,
            border: `1px solid ${!selectedOrg ? PRIMARY : G300}`,
            fontSize: 11, fontWeight: 600, cursor: "pointer",
            backgroundColor: !selectedOrg ? PRIMARY : WHITE,
            color: !selectedOrg ? WHITE : G700,
          }}>All</button>
        {activeOrgs.map((org) => (
          <button key={org} onClick={() => { setSelectedOrg(org); setSelectedApp(null); }}
            style={{
              padding: "4px 12px", borderRadius: 5,
              border: `1px solid ${selectedOrg === org ? PRIMARY : G300}`,
              fontSize: 11, fontWeight: 600, cursor: "pointer",
              backgroundColor: selectedOrg === org ? PRIMARY : WHITE,
              color: selectedOrg === org ? WHITE : G700,
            }}>{org}</button>
        ))}

        {selectedOrg && availableApps.length > 0 && (
          <>
            <div style={{ width: 1, height: 20, backgroundColor: G300, margin: "0 4px" }} />
            <span style={{ fontSize: 10, fontWeight: 700, color: G400, textTransform: "uppercase", letterSpacing: "0.08em", marginRight: 4 }}>
              App
            </span>
            <button onClick={() => setSelectedApp(null)}
              style={{
                padding: "4px 10px", borderRadius: 5,
                border: `1px solid ${!selectedApp ? PRIMARY : G300}`,
                fontSize: 11, fontWeight: 500, cursor: "pointer",
                backgroundColor: !selectedApp ? PRIMARY : WHITE,
                color: !selectedApp ? WHITE : G700,
              }}>All</button>
            {availableApps.map((app) => (
              <button key={app} onClick={() => setSelectedApp(app)}
                style={{
                  padding: "4px 10px", borderRadius: 5,
                  border: `1px solid ${selectedApp === app ? PRIMARY : G300}`,
                  fontSize: 10, fontWeight: 500, cursor: "pointer",
                  backgroundColor: selectedApp === app ? PRIMARY : WHITE,
                  color: selectedApp === app ? WHITE : G700,
                }}>{app}</button>
            ))}
          </>
        )}
      </div>

      {/* MAIN CONTENT */}
      <div style={{ padding: "24px 32px" }}>

        <div style={{ fontSize: 12, fontWeight: 500, color: G500, marginBottom: 16 }}>
          Showing: <span style={{ fontWeight: 700, color: G900 }}>
            {selectedApp ? `${selectedOrg} → ${selectedApp}` : selectedOrg || "All Org Units"}
          </span>
          <span style={{ marginLeft: 8, color: G400 }}>|</span>
          <span style={{ marginLeft: 8, color: G500 }}>{timeRange.charAt(0).toUpperCase() + timeRange.slice(1)} view</span>
        </div>

        <div style={{ display: "flex", gap: 16, alignItems: "stretch" }}>
          {ROLES.map((role) => (
            <RoleColumn key={role} role={role} data={roleData} />
          ))}
        </div>

        {!selectedOrg && (
          <div style={{
            marginTop: 24, backgroundColor: WHITE, borderRadius: 10,
            border: `1px solid ${G200}`, padding: "16px 24px",
          }}>
            <div style={{
              fontSize: 11, fontWeight: 700, color: G500, textTransform: "uppercase",
              letterSpacing: "0.06em", marginBottom: 14,
            }}>
              Acceleration by Org Unit × Role
            </div>
            <table style={{ width: "100%", borderCollapse: "collapse", fontSize: 12 }}>
              <thead>
                <tr>
                  <th style={{ textAlign: "left", padding: "6px 12px", fontSize: 10, fontWeight: 700, color: G400, textTransform: "uppercase" }}>Org Unit</th>
                  {ROLES.map((r) => (
                    <th key={r} style={{
                      textAlign: "center", padding: "6px 12px", fontSize: 10, fontWeight: 700,
                      color: ROLE_ACCENT[r].text, textTransform: "uppercase",
                    }}>{r}</th>
                  ))}
                </tr>
              </thead>
              <tbody>
                {orgRoleAccel.map((row) => (
                  <tr key={row.org} style={{ borderTop: `1px solid ${G100}` }}>
                    <td style={{ padding: "10px 12px", fontWeight: 600, color: G900 }}>{row.org}</td>
                    {ROLES.map((r) => (
                      <td key={r} style={{ textAlign: "center", padding: "10px 12px" }}>
                        <div style={{
                          display: "inline-block", padding: "3px 12px", borderRadius: 4,
                          backgroundColor: ROLE_ACCENT[r].light,
                          fontWeight: 700, fontSize: 14, color: ROLE_ACCENT[r].text,
                        }}>
                          {row[r]}%
                        </div>
                      </td>
                    ))}
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        )}

        <div style={{ textAlign: "center", padding: "20px 0 4px", fontSize: 10, color: G400 }}>
          v0.3 Prototype — Mock Data — For Executive Review & Feedback
        </div>
      </div>
    </div>
  );
}
