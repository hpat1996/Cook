import random
from collections import Counter
from typing import Dict, List, Tuple
import streamlit as st
import pandas as pd
import altair as alt

class ReceiptGenerator:
    def __init__(self, items_prices: Dict[str, float]):
        self.items_prices = items_prices

    def generate_receipts(self, total_amount: float, total_receipts: int) -> Tuple[List[List[Tuple[str, int, float]]], float]:
        items = list(self.items_prices.keys())
        receipts = []

        total_all = 0
        all_items_ordered = []

        while total_all < total_amount:
            item = random.choice(items)
            price = self.items_prices[item]

            all_items_ordered.append(item)
            total_all += price

        for i in range(total_receipts):
            receipts.append([])

        for item in all_items_ordered:
            receipt = random.choice(receipts)
            receipt.append(item)

        for i in range(len(receipts)):
            receipt = receipts[i]
            receipt = Counter(receipt)
            receipts[i] = [(item, qty, self.items_prices[item]) for item, qty in receipt.items()]

        return receipts, total_all


def main():
    st.title("Receipt Generator")

    if "items_prices" not in st.session_state:
        st.session_state.items_prices = {}

    with st.container():
        st.subheader("Items and Prices")
        df = pd.DataFrame([(item, price) for item, price in st.session_state.items_prices.items()],
                            columns=["Item", "Price"])

        # Make the dataframe editable
        edited_df = st.data_editor(
            df,
            num_rows="dynamic",
            column_config={
                "Item": st.column_config.TextColumn(
                    "Item",
                    help="Enter the name of the item",
                    max_chars=200,
                    validate="^[A-Za-z0-9 ]+",
                    width=200,
                ),
                "Price": st.column_config.NumberColumn(
                    "Price",
                    help="Enter the price of the item",
                    min_value=1,
                    max_value=1000,
                    step=1,
                    width=100,
                )
            },
            hide_index=True,
        )

        # Update the session state with the edited data
        st.session_state.items_prices = dict(zip(edited_df["Item"], edited_df["Price"]))

    total_amount = st.number_input('Total target amount (Rs)', min_value=1, step=1)
    total_receipts = st.number_input('Number of receipts', min_value=1, step=1)

    if st.button('Generate Receipts'):
        if not st.session_state.items_prices:
            st.error("Please add some items before generating receipts.")
        else:
            generator = ReceiptGenerator(st.session_state.items_prices)
            with st.spinner("Generating receipts..."):
                receipts, total_all = generator.generate_receipts(total_amount, total_receipts)

            st.success(f"Generated {len(receipts)} receipts with a total of Rs {int(total_all)}")

            for i, receipt in enumerate(receipts, 1):
                with st.expander(f"Receipt {i}"):
                    df = pd.DataFrame(receipt, columns=["Item", "Quantity", "Price"])
                    df["Total"] = df["Price"] * df["Quantity"]
                    df = df[["Item", "Quantity", "Price", "Total"]]
                    st.dataframe(df)
                    st.write(f"Receipt Total: Rs {int(df['Total'].sum())}")

            # Visualization
            all_items = Counter()
            for receipt in receipts:
                all_items.update(dict([(item, qty) for item, qty, _ in receipt]))

            chart_data = pd.DataFrame.from_dict(all_items, orient='index', columns=['Quantity']).reset_index()
            chart_data.columns = ['Item', 'Quantity']

            chart = alt.Chart(chart_data).mark_bar().encode(
                x='Item',
                y='Quantity',
                color='Item'
            ).properties(
                title='Total Quantity of Items Across All Receipts'
            )

            st.altair_chart(chart, use_container_width=True)

if __name__ == "__main__":
    main()
