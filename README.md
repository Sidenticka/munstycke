import cadquery as cq

# --- Parametrar ---
L = 220.0
D_outer = 14.0
D_inlet = 2.5
D_vent = 4.5
D_outlet = 1.8
x_row1 = 110.0
x_row2 = 118.0

# 1. Skapa huvudkroppen (Stålaxeln)
# Vi skapar en cylinder och lägger till M8-gängans ytterdiameter (8mm) i fronten
result = (
    cq.Workplane("XY")
    .circle(D_outer / 2).extrude(L)
)

# Svarva ner fronten för M8-gänga (15mm) och släppning (3mm)
result = (
    result.faces("<Z").workplane()
    .circle(D_outer / 2 + 1).rect(20, 20).cutBlind(-15) # Rensa utsidan
    .circle(8.0 / 2).extrude(15, combine='join') # Lägg till M8-cylindern
)

# 2. Skapa den inre kanalen (Hålprofilen)
# Vi ritar halva tvärsnittet och roterar det (Revolve)
inner_profile = (
    cq.Workplane("XZ")
    .lineTo(0, D_inlet / 2)
    .lineTo(35, D_inlet / 2)
    .lineTo(50, D_vent / 2)   # Expansion (Avgasning start)
    .lineTo(75, D_vent / 2)   # Vakuumplatå
    .lineTo(90, D_inlet / 2)  # Rekompression
    .lineTo(110, D_inlet / 2) # Fram till fiberinlopp
    .lineTo(180, D_outlet / 2)# Konisk del (Mjuk radie-ish)
    .lineTo(L, D_outlet / 2)
    .lineTo(L, 0)
    .close()
)

# Skär ut kanalen från solidsen
result = result.cut(inner_profile.revolve(360, (0, 0, 0), (1, 0, 0)))

# 3. Borra de 28 fiberhålen (Sicksack)
# Rad 1 (14 hål vid x=110)
for i in range(14):
    angle = i * (360 / 14)
    result = (
        result.workplane("XY")
        .center(x_row1, 0).rotateAboutCenter((1, 0, 0), angle)
        .move(0, D_outer / 2).circle(0.125).cutBlind(-8) # 0.25mm hål
    )

# Rad 2 (14 hål vid x=118, förskjutna 12.8 grader)
for i in range(14):
    angle = i * (360 / 14) + 12.8
    result = (
        result.workplane("XY")
        .center(x_row2, 0).rotateAboutCenter((1, 0, 0), angle)
        .move(0, D_outer / 2).circle(0.125).cutBlind(-8)
    )

# 4. Lägg till Vakuumporten (6mm hål vid x=62.5)
result = (
    result.faces(">Y").workplane()
    .center(62.5, 0).circle(3.0).cutThruAll()
)

# --- EXPORT ---
# Denna rad skapar filen på din dator om du kör skriptet lokalt
cq.exporters.export(result, 'extrudermunstycke_v2_1.step')
