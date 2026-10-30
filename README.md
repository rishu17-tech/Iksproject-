import streamlit as st
import math
import time
from io import StringIO
from contextlib import redirect_stdout


# -------------------------------
# PAGE CONFIGURATION
# -------------------------------

st.set_page_config(
    page_title="Aryabhatiya Root Calculator",
    page_icon="📐",
    layout="wide"
)

st.title("📐 Aryabhatiya Root Calculator")
st.subheader("Traditional Mathematics vs Modern Computing")

st.markdown("""
This application compares Aryabhata-style traditional root extraction
with modern computational techniques for calculating square roots
and cube roots.
""")

st.divider()


# -------------------------------
# GROUP DIGITS
# -------------------------------

def make_groups(n, size):
    s = str(n)
    first = len(s) % size

    groups = [int(s[:first])] if first else []

    groups += [
        int(s[i:i + size])
        for i in range(first, len(s), size)
    ]

    return groups


# -------------------------------
# ARYABHATA SQUARE ROOT
# -------------------------------

def aryabhata_sqrt(n, show=False):

    if n < 0:
        raise ValueError("Number must be non-negative")

    groups = make_groups(n, 2) if n else [0]

    if show:
        print("Step 1: Divide digits into pairs from right.")
        print("Groups:", groups)

    root, rem = 0, 0

    for i, g in enumerate(groups, 1):

        cur = rem * 100 + g

        for d in range(9, -1, -1):
            if (20 * root + d) * d <= cur:
                break

        sub = (20 * root + d) * d

        if show:

            if i == 1:
                print(f"\nStep 2: First group = {g}")
                print(f"Largest square = {d}² = {d*d}")

            else:
                print(f"\nStep {i+1}: Bring down {g:02d}")
                print(f"Current value = {cur}")
                print(f"Selected digit = {d}")
                print(f"Subtract = {sub}")

            print(f"Remainder = {cur-sub}")
            print(f"Root so far = {10*root+d}")

        rem = cur - sub
        root = 10 * root + d

    return root, rem


# -------------------------------
# ARYABHATA CUBE ROOT
# -------------------------------

def aryabhata_cbrt(n, show=False):

    if n < 0:
        raise ValueError("Number must be non-negative")

    groups = make_groups(n, 3) if n else [0]

    if show:
        print("Step 1: Divide digits into groups of three.")
        print("Groups:", groups)

    root = max(
        i for i in range(10)
        if i ** 3 <= groups[0]
    )

    rem = groups[0] - root ** 3

    if show:
        print(f"\nFirst group = {groups[0]}")
        print(f"Largest cube = {root}³ = {root**3}")
        print(f"Remainder = {rem}")

    for i, g in enumerate(groups[1:], 3):

        cur = rem * 1000 + g
        d = 0

        while d < 9:

            nxt = d + 1

            value = (
                300 * root**2 * nxt
                + 30 * root * nxt**2
                + nxt**3
            )

            if value <= cur:
                d = nxt
            else:
                break

        t1 = 300 * root**2 * d
        t2 = 30 * root * d**2
        t3 = d**3

        total = t1 + t2 + t3

        if show:

            print(f"\nStep {i}: Bring down {g:03d}")
            print(f"Current value = {cur}")
            print(f"Selected digit = {d}")

            print(f"3r²d = {t1}")
            print(f"3rd² = {t2}")
            print(f"d³ = {t3}")

            print(f"Total subtraction = {total}")
            print(f"Remainder = {cur-total}")
            print(f"Root so far = {10*root+d}")

        rem = cur - total
        root = 10 * root + d

    return root, rem


# -------------------------------
# MODERN SQUARE ROOT
# -------------------------------

def newton_sqrt(n, show=False):

    if n < 2:
        return n, 0

    x = n
    y = (x + 1) // 2
    it = 0

    while y < x:

        x, y = y, (y + n // y) // 2
        it += 1

        if show:
            print(f"Iteration {it}: x = {x}")

    return x, it


# -------------------------------
# MODERN CUBE ROOT
# -------------------------------

def modern_cbrt(n):

    r = round(n ** (1 / 3))

    while r ** 3 > n:
        r -= 1

    while (r + 1) ** 3 <= n:
        r += 1

    return r


# -------------------------------
# EXECUTION TIME
# -------------------------------

def timeit(func, arg, repeat=500):

    start = time.perf_counter()

    for _ in range(repeat):
        func(arg)

    return (
        (time.perf_counter() - start)
        / repeat * 1e6
    )


# -------------------------------
# REPORT GENERATION
# -------------------------------

def report(n, kind):

    line = "=" * 55

    name = (
        "SQUARE ROOT"
        if kind == "sqrt"
        else "CUBE ROOT"
    )

    print(line)
    print(f"{name} OF {n}")
    print(line)

    print("\n[A] ARYABHATA TRADITIONAL METHOD")

    if kind == "sqrt":

        root, rem = aryabhata_sqrt(n, show=True)

    else:

        root, rem = aryabhata_cbrt(n, show=True)

    print(f"\nTraditional Result = {root}")
    print(f"Remainder = {rem}")

    print("\n[B] MODERN COMPUTING METHOD")

    if kind == "sqrt":

        m = math.isqrt(n)

        print(f"math.isqrt({n}) = {m}")
        print(f"math.sqrt({n}) = {math.sqrt(n):.6f}")

        print("\nNewton-Raphson Method:")

        nr, its = newton_sqrt(n, show=True)

        print(f"Newton Result = {nr}")
        print(f"Iterations = {its}")

        t_old = timeit(
            lambda x: aryabhata_sqrt(x), n
        )

        t_new = timeit(math.isqrt, n)

    else:

        m = modern_cbrt(n)

        print(f"Modern Cube Root = {m}")
        print(f"Floating Point = {n ** (1/3):.6f}")

        t_old = timeit(
            lambda x: aryabhata_cbrt(x), n
        )

        t_new = timeit(modern_cbrt, n)

    print("\n[C] PERFORMANCE COMPARISON")

    print(f"{'Method':<22}{'Result':>10}{'Time (µs)':>15}")

    print(
        f"{'Aryabhata Method':<22}"
        f"{root:>10}"
        f"{t_old:>15.2f}"
    )

    print(
        f"{'Modern Method':<22}"
        f"{m:>10}"
        f"{t_new:>15.2f}"
    )

    print("\nVerification:", "MATCH" if root == m else "MISMATCH")


# -------------------------------
# STREAMLIT USER INTERFACE
# -------------------------------

st.header("🔢 Root Calculator")

col1, col2 = st.columns(2)

with col1:

    method = st.selectbox(
        "Select Calculation Method",
        ["Square Root", "Cube Root"]
    )

with col2:

    number = st.text_input(
        "Enter a Non-Negative Integer",
        value="144"
    )

show_steps = st.checkbox(
    "Show Traditional Calculation Steps",
    value=True
)

calculate = st.button(
    "Calculate Root 🚀",
    use_container_width=True
)


# -------------------------------
# CALCULATION OUTPUT
# -------------------------------

if calculate:

    try:

        n = int(number.strip())

        if n < 0:
            st.error("Please enter a non-negative integer.")

        elif n > 10**100:
            st.error("Please enter a number with at most 100 digits.")

        else:

            kind = (
                "sqrt"
                if method == "Square Root"
                else "cbrt"
            )

            output = StringIO()

            with redirect_stdout(output):

                report(n, kind)

            result_text = output.getvalue()

            if kind == "sqrt":
                root, remainder = aryabhata_sqrt(n)
            else:
                root, remainder = aryabhata_cbrt(n)

            st.success("Calculation Completed Successfully!")

            st.subheader("📊 Calculation Result")

            c1, c2, c3 = st.columns(3)

            with c1:
                st.metric("Integer Root", root)

            with c2:
                st.metric("Remainder", remainder)

            with c3:
                st.metric(
                    "Verification",
                    "MATCH" if (
                        root == (
                            math.isqrt(n)
                            if kind == "sqrt"
                            else modern_cbrt(n)
                        )
                    ) else "MISMATCH"
                )

            st.divider()

            st.subheader("📜 Detailed Calculation Report")

            st.code(
                result_text,
                language="text"
            )

    except ValueError:
        st.error("Please enter a valid non-negative integer.")


# -------------------------------
# SAMPLE TESTS
# -------------------------------

st.divider()

st.header("🧪 Sample Test Cases")

if st.button("Run Sample Tests"):

    st.subheader("Square Root Tests")

    for n in [144, 625, 112896, 999999]:

        root, rem = aryabhata_sqrt(n)

        st.write(
            f"√{n} = {root}, Remainder = {rem}"
        )

    st.subheader("Cube Root Tests")

    for n in [125, 1953125, 76928302277]:

        root, rem = aryabhata_cbrt(n)

        st.write(
            f"∛{n} = {root}, Remainder = {rem}"
        )


# -------------------------------
# FOOTER
# -------------------------------

st.divider()

st.markdown(
    """
    <div style="text-align:center">
        <h4>Aryabhatiya Root Calculator</h4>
        <p>Traditional Indian Mathematics Meets Modern Technology</p>
        <p>Project Demonstration</p>
    </div>
    """,
    unsafe_allow_html=True
)
