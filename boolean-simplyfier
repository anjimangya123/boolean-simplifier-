"""Scalable Boolean Logic Minimizer - Quine-McCluskey dashboard (Tkinter, no extra installs).

Run:  python qm_dashboard.py
"""
import re
import tkinter as tk
from tkinter import ttk, messagebox


# ----------------------------------------------------------------- core logic
def letter(i):
    return chr(65 + i)


def parse_list(text, n):
    out = []
    for tok in re.split(r"[,\s]+", text.strip()):
        if not tok:
            continue
        if not tok.isdigit():
            raise ValueError(f"'{tok}' is not a whole number.")
        v = int(tok)
        if v >= 2 ** n:
            raise ValueError(f"Minterm {v} is too large for {n} variables (max {2 ** n - 1}).")
        out.append(v)
    return sorted(set(out))


def parse_expr(text):
    s = text.replace("\u2032", "'").replace(" ", "")
    if not s:
        raise ValueError("Enter an expression.")
    terms = []
    for t in s.split("+"):
        if not re.fullmatch(r"(?:!?[A-Za-z]'?)+", t or "#"):
            raise ValueError(f"Cannot read term '{t}'.")
        lits = [(ord(m[1].upper()) - 65, not (m[0] or m[2]))
                for m in re.findall(r"(!?)([A-Za-z])('?)", t)]
        terms.append(lits)
    n = max(i for t in terms for i, _ in t) + 1
    if n > 10:
        raise ValueError("Use variables A-J only.")
    return terms, max(n, 2)


def expr_to_minterms(terms, n):
    out = []
    for v in range(2 ** n):
        bits = format(v, f"0{n}b")
        if any(all((bits[i] == "1") == pos for i, pos in t) for t in terms):
            out.append(v)
    return out


def combine(a, b):
    diff = [i for i in range(len(a)) if a[i] != b[i]]
    if len(diff) != 1 or "-" in (a[diff[0]], b[diff[0]]):
        return None
    d = diff[0]
    return a[:d] + "-" + a[d + 1:]


def quine_mccluskey(minterms, dcs, n):
    cur = {format(v, f"0{n}b"): {v} for v in set(minterms) | set(dcs)}
    steps, primes = [], []
    while cur:
        groups = {}
        for k in cur:
            groups.setdefault(k.count("1"), []).append(k)
        steps.append([(g, [(k, sorted(cur[k])) for k in groups[g]]) for g in sorted(groups)])
        used, nxt = set(), {}
        for g in sorted(groups):
            for a in groups.get(g, []):
                for b in groups.get(g + 1, []):
                    c = combine(a, b)
                    if c:
                        used |= {a, b}
                        nxt.setdefault(c, set()).update(cur[a], cur[b])
        primes += [(k, sorted(v)) for k, v in cur.items() if k not in used]
        cur = nxt

    need = set(minterms)
    chosen, essential = set(), set()
    for m in minterms:
        cover = [p for p in primes if m in p[1]]
        if len(cover) == 1:
            chosen.add(cover[0][0])
            essential.add(cover[0][0])
    covered = {m for k, c in primes if k in chosen for m in c}
    while need - covered:  # greedy cover for what is left
        best = max((p for p in primes if p[0] not in chosen),
                   key=lambda p: len((need - covered) & set(p[1])))
        chosen.add(best[0])
        covered |= set(best[1])
    return steps, primes, chosen, essential


def term_str(bits):
    s = "".join(letter(i) + ("\u2032" if b == "0" else "") for i, b in enumerate(bits) if b != "-")
    return s or "1"


def literal_count(bits):
    return sum(b != "-" for b in bits)


# ------------------------------------------------------------------------ GUI
INK, PANEL, BG, ACC, HOT = "#14213d", "#ffffff", "#eef1f4", "#0f766e", "#c2410c"


class App(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Boolean Logic Minimizer - Quine-McCluskey")
        self.geometry("1120x700")
        self.minsize(900, 560)
        self.configure(bg=BG)
        st = ttk.Style(self)
        st.theme_use("clam")
        st.configure("TFrame", background=BG)
        st.configure("Panel.TFrame", background=PANEL)
        st.configure("TLabel", background=BG, foreground=INK, font=("Segoe UI", 10))
        st.configure("Panel.TLabel", background=PANEL)
        st.configure("Card.TLabel", background=PANEL, foreground=ACC, font=("Segoe UI", 22, "bold"))
        st.configure("CardCap.TLabel", background=PANEL, foreground="#5b6577", font=("Segoe UI", 9))
        st.configure("TButton", font=("Segoe UI", 10, "bold"), padding=7)
        st.configure("Treeview", rowheight=24, font=("Consolas", 10))
        st.configure("Treeview.Heading", font=("Segoe UI", 9, "bold"))
        self._build()
        self.minimize()

    def _build(self):
        side = ttk.Frame(self, padding=14, style="Panel.TFrame")
        side.pack(side="left", fill="y", padx=(12, 6), pady=12)
        ttk.Label(side, text="Input", style="Panel.TLabel", font=("Segoe UI", 15, "bold")).pack(anchor="w")

        self.mode = tk.StringVar(value="m")
        for text, val in (("Minterms", "m"), ("Expression (SOP)", "e")):
            ttk.Radiobutton(side, text=text, value=val, variable=self.mode, command=self._toggle).pack(anchor="w")

        self.n = tk.IntVar(value=4)
        ttk.Label(side, text="Variables (2-10)", style="Panel.TLabel").pack(anchor="w", pady=(12, 0))
        ttk.Spinbox(side, from_=2, to=10, textvariable=self.n, width=6).pack(anchor="w")

        self.mbox = ttk.Frame(side, style="Panel.TFrame")
        self.mbox.pack(fill="x")
        ttk.Label(self.mbox, text="Minterms", style="Panel.TLabel").pack(anchor="w", pady=(12, 0))
        self.mt = ttk.Entry(self.mbox, width=34)
        self.mt.insert(0, "0,1,2,5,6,7,8,9,10,14")
        self.mt.pack(anchor="w")
        ttk.Label(self.mbox, text="Don't-cares (optional)", style="Panel.TLabel").pack(anchor="w", pady=(8, 0))
        self.dc = ttk.Entry(self.mbox, width=34)
        self.dc.pack(anchor="w")

        self.ebox = ttk.Frame(side, style="Panel.TFrame")
        ttk.Label(self.ebox, text="Expression", style="Panel.TLabel").pack(anchor="w", pady=(12, 0))
        self.ex = ttk.Entry(self.ebox, width=34)
        self.ex.insert(0, "A'B' + A'C + BC' + AB'")
        self.ex.pack(anchor="w")
        ttk.Label(self.ebox, text="Use ' or ! for NOT, + for OR", style="Panel.TLabel",
                  foreground="#5b6577").pack(anchor="w")

        self.run_btn = ttk.Button(side, text="Minimize", command=self.minimize)
        self.run_btn.pack(fill="x", pady=(18, 4))
        ttk.Button(side, text="6-variable example", command=self.example).pack(fill="x", pady=4)
        ttk.Button(side, text="Clear", command=self.clear).pack(fill="x", pady=4)
        self.status = ttk.Label(side, text="", style="Panel.TLabel", foreground=HOT, wraplength=250)
        self.status.pack(anchor="w", pady=8)

        main = ttk.Frame(self)
        main.pack(side="left", fill="both", expand=True, padx=(6, 12), pady=12)

        self.answer = tk.Label(main, text="F = ", font=("Consolas", 18, "bold"), bg="#fde9c8", fg=INK,
                               anchor="w", padx=14, pady=12, relief="flat")
        self.answer.pack(fill="x")

        cards = ttk.Frame(main)
        cards.pack(fill="x", pady=10)
        self.card = {}
        for key, cap in (("minterms", "Minterms"), ("primes", "Prime implicants"),
                         ("essential", "Essential"), ("terms", "Terms in answer"), ("literals", "Literals")):
            f = ttk.Frame(cards, style="Panel.TFrame", padding=10)
            f.pack(side="left", fill="x", expand=True, padx=3)
            v = ttk.Label(f, text="-", style="Card.TLabel")
            v.pack(anchor="w")
            ttk.Label(f, text=cap, style="CardCap.TLabel").pack(anchor="w")
            self.card[key] = v

        tabs = ttk.Notebook(main)
        tabs.pack(fill="both", expand=True)
        self.steps_tv = self._tree(tabs, "Grouping steps", ("col", "ones", "minterms", "bits"),
                                   ("Column", "1s", "Minterms", "Bits"), (70, 50, 260, 160))
        self.chart_tv = self._tree(tabs, "Prime implicant chart", ("term",), ("Term",), (110,))

    def _tree(self, tabs, title, cols, heads, widths):
        frame = ttk.Frame(tabs)
        tabs.add(frame, text=title)
        tv = ttk.Treeview(frame, columns=cols, show="headings")
        for c, h, w in zip(cols, heads, widths):
            tv.heading(c, text=h)
            tv.column(c, width=w, anchor="center")
        ys = ttk.Scrollbar(frame, orient="vertical", command=tv.yview)
        xs = ttk.Scrollbar(frame, orient="horizontal", command=tv.xview)
        tv.configure(yscrollcommand=ys.set, xscrollcommand=xs.set)
        ys.pack(side="right", fill="y")
        xs.pack(side="bottom", fill="x")
        tv.pack(fill="both", expand=True)
        tv.tag_configure("ess", foreground=HOT, font=("Consolas", 10, "bold"))
        tv.tag_configure("chosen", foreground=ACC)
        return tv

    def _toggle(self):
        if self.mode.get() == "e":
            self.mbox.pack_forget()
            self.ebox.pack(fill="x", before=self.run_btn)
        else:
            self.ebox.pack_forget()
            self.mbox.pack(fill="x", before=self.run_btn)

    def clear(self):
        for e in (self.mt, self.dc, self.ex):
            e.delete(0, "end")
        self.status.config(text="")

    def example(self):
        self.mode.set("m")
        self._toggle()
        self.n.set(6)
        self.mt.delete(0, "end")
        self.mt.insert(0, "1,3,5,7,9,11,13,15,17,19,21,23,33,35,37,39,41,43,45,47,60,61")
        self.dc.delete(0, "end")
        self.dc.insert(0, "0,2")
        self.minimize()

    def minimize(self):
        self.status.config(text="")
        try:
            n = int(self.n.get())
            if not 2 <= n <= 10:
                raise ValueError("Choose between 2 and 10 variables.")
            dcs = []
            if self.mode.get() == "e":
                terms, dn = parse_expr(self.ex.get())
                n = max(n, dn)
                self.n.set(n)
                minterms = expr_to_minterms(terms, n)
            else:
                minterms = parse_list(self.mt.get(), n)
                dcs = parse_list(self.dc.get(), n)
                if set(minterms) & set(dcs):
                    raise ValueError("A number cannot be both a minterm and a don't-care.")
            self._show(minterms, dcs, n)
        except ValueError as e:
            self.status.config(text=str(e))

    def _show(self, minterms, dcs, n):
        for tv in (self.steps_tv, self.chart_tv):
            tv.delete(*tv.get_children())
        if not minterms:
            self.answer.config(text="F = 0")
            for c in self.card.values():
                c.config(text="0")
            return
        steps, primes, chosen, essential = quine_mccluskey(minterms, dcs, n)
        sel = [p for p in primes if p[0] in chosen]
        self.answer.config(text="F = " + " + ".join(term_str(k) for k, _ in sel))
        vals = {"minterms": len(minterms), "primes": len(primes), "essential": len(essential),
                "terms": len(sel), "literals": sum(literal_count(k) for k, _ in sel)}
        for k, v in vals.items():
            self.card[k].config(text=str(v))

        for i, col in enumerate(steps, 1):
            for ones, rows in col:
                for bits, cov in rows:
                    self.steps_tv.insert("", "end", values=(i, ones, ",".join(map(str, cov)), bits))

        cols = ["term"] + [f"m{m}" for m in minterms] + ["status"]
        self.chart_tv.configure(columns=cols)
        self.chart_tv.heading("term", text="Term")
        self.chart_tv.column("term", width=110, anchor="center")
        for m in minterms:
            self.chart_tv.heading(f"m{m}", text=str(m))
            self.chart_tv.column(f"m{m}", width=42, anchor="center", stretch=False)
        self.chart_tv.heading("status", text="Status")
        self.chart_tv.column("status", width=90, anchor="center", stretch=False)
        for bits, cov in primes:
            tag = "ess" if bits in essential else "chosen" if bits in chosen else ""
            status = "essential" if bits in essential else "chosen" if bits in chosen else ""
            row = [f"{term_str(bits)}  ({bits})"] + ["\u25cf" if m in cov else "" for m in minterms] + [status]
            self.chart_tv.insert("", "end", values=row, tags=(tag,))


if __name__ == "__main__":
    App().mainloop()