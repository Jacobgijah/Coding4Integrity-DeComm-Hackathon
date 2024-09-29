<script>
    import { actorBackend } from "$lib/motokoImports/backend";
    import { onMount } from "svelte";

    let balanceAmount = 0;
    let buyingPower = 580;
    let activities = []; // Transaction activities
    let modalVisible = false;
    let modalTitle = '';
    let address = '';
    let amount = '';
    let searchFilter = '';
    let loading = true; // Loading state

    // Fetch wallet balance and transactions for the logged-in user
    onMount(async () => {
        try {
            const userName = sessionStorage.getItem("userName");
            console.log("Retrieved userName:", userName);

            if (!userName) {
                throw new Error("User not logged in");
            }

            // Fetch wallet balance
            const wallet = await actorBackend.getTotalWalletValue(userName);
            balanceAmount = wallet.reduce((total, price) => total + Number(price.amount), 0);

            // Fetch transaction activities for the user
            const transactions = await actorBackend.getUserTransactions(userName);

            // Map transactions to the activities array for display
            activities = transactions.map((tx) => {
                return {
                    productID: tx.productID,
                    buyerID: tx.buyerID,
                    amount: Number(tx.paidPrice.amount),  // Convert Nat to number
                    date: new Date(Number(tx.id)).toLocaleDateString('en-US', {
                        year: 'numeric',
                        month: 'short',
                        day: 'numeric',
                    }),
                    currency: tx.paidPrice.currency
                };
            });

            console.log("Fetched user transaction activities:", activities);  // Debugging

        } catch (error) {
            console.error("Error fetching data:", error);
        } finally {
            loading = false;
        }
    });

    const showModal = (type) => {
        modalTitle = type;
        modalVisible = true;
        setTimeout(() => {
            document.querySelector('.modal input').focus(); // Focus on the first input
        }, 0);
    };

    const submitTransaction = () => {
        if (address.trim() === '' || amount <= 0) {
            alert('Please enter a valid address and amount.');
            return;
        }

        const date = new Date().toLocaleDateString('en-US', { year: 'numeric', month: 'short', day: 'numeric' });
        activities = [{ type: modalTitle, address, amount: parseFloat(amount), date }, ...activities];

        balanceAmount += modalTitle === "Deposit" ? parseFloat(amount) : -parseFloat(amount);
        resetModal();
    };

    const resetModal = () => {
        modalVisible = false;
        address = '';
        amount = '';
    };
</script>

<style>
    .body {
        background-color: #f0f0f0;
        font-family: 'Arial', sans-serif;
        margin: 0;
        margin-top: 57px;
        padding: 0;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
    }
    .card {
        background-color: #fff;
        color: #000;
        width: 400px;
        padding: 20px;
        border-radius: 12px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
    }
    .header {
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .header h1 {
        font-size: 24px;
    }
    .header .search-icon {
        font-size: 20px;
        cursor: pointer;
        transition: transform 0.3s;
    }
    .balance {
        margin-top: 20px;
    }
    .balance h2 {
        font-size: 20px;
    }
    .buying-power {
        margin-top: 20px;
        display: flex;
        justify-content: space-between;
        align-items: center;
    }
    .buttons button {
        background-color: transparent;
        border: 1px solid #000;
        color: #000;
        padding: 10px 20px;
        margin-right: 10px;
        cursor: pointer;
        border-radius: 12px;
        transition: background-color 0.3s, color 0.3s;
    }
    .buttons button:hover {
        background-color: #333131;
        color: #fff;
    }
    .recent-activity {
        margin-top: 40px;
        max-height: 200px;
        overflow-y: auto;
        border: 1px solid #000;
        padding: 10px;
        border-radius: 12px;
    }
    .recent-activity h2 {
        font-size: 16px;
        margin: 0 0 20px 0;
    }
    .activity-item {
        display: flex;
        justify-content: space-between;
        margin-bottom: 10px;
    }
    .activity-item .details {
        font-size: 14px;
    }
    .activity-item .amount {
        font-size: 14px;
    }
    .modal {
        display: block; /* Always display the modal but hide it visually */
        position: fixed;
        z-index: 1;
        left: 0;
        top: 0;
        width: 100%;
        height: 100%;
        overflow: auto;
        margin-top: 57px;
        background-color: rgba(0, 0, 0, 0.4);
        display: none; /* Hide modal initially */
    }
    .modal.visible {
        display: block; /* Show modal when it has the 'visible' class */
        animation: fadeIn 0.5s;
    }
    .modal-content {
        background-color: #fff;
        margin: 5% auto;
        padding: 20px;
        border: 1px solid #888;
        width: 50%;
        color: #000;
        border-radius: 12px;
        animation: slideIn 0.5s;
    }
    .close {
        color: #aaa;
        float: right;
        font-size: 28px;
        font-weight: bold;
    }
    .close:hover,
    .close:focus {
        color: #000;
        cursor: pointer;
    }
    .modal input {
        width: 100%;
        padding: 10px;
        margin: 10px 0;
        box-sizing: border-box;
        border: 1px solid #000;
        border-radius: 12px;
        background-color: #ccc;
    }
    .modal button {
        background-color: #000;
        color: #fff;
        padding: 10px 20px;
        border: none;
        cursor: pointer;
        border-radius: 12px;
    }
    @keyframes fadeIn {
        from { opacity: 0; }
        to { opacity: 1; }
    }
    @keyframes slideIn {
        from { transform: translateY(-50px); }
        to { transform: translateY(0); }
    }
</style>

<div class="body">
    <div class="card">
        <div class="header">
            <h1>Wallet</h1>
            <i class="fas fa-search search-icon" on:click={() => searchFilter = ''}></i>
            <input type="text" class="search-bar" placeholder="Search transactions..." bind:value={searchFilter}>
        </div>

        <div class="balance">
            <h2>Balance</h2>
            {#if loading}
                <div>Loading...</div>
            {:else}
                <div class="amount" style="font-size: 22px;">{balanceAmount.toLocaleString()} KT</div>
            {/if}
        </div>

        <div class="buying-power">
            <h2>Buying Power</h2>
            <div class="amount">{buyingPower.toFixed(2)} KT</div>
        </div>

        <div class="buttons">
            <button on:click={() => showModal('Deposit')}>Deposit</button>
            <button on:click={() => showModal('Withdraw')}>Withdraw</button>
        </div>

        <!-- Recent Activity Section -->
        <div class="recent-activity">
            <h2>Recent Activity</h2>
            {#if activities.length > 0}
                {#each activities as activity (activity.date)}
                    <div class="activity-item">
                        <div class="details">
                            Product {activity.productID} - Buyer: {activity.buyerID} on {activity.date} ({activity.currency})
                        </div>
                        <div class="amount">{activity.amount.toFixed(4)} {activity.currency}</div>
                    </div>
                {/each}
            {:else}
                <div>No recent activity</div>
            {/if}
        </div>
    </div>

    <!-- Modal for Deposit/Withdraw -->
    <div class={`modal ${modalVisible ? 'visible' : ''}`}>
        <div class="modal-content">
            <span class="close" on:click={resetModal}>&times;</span>
            <h2>{modalTitle}</h2>
            <input type="text" placeholder="Address" bind:value={address} />
            <input type="number" placeholder="Amount" bind:value={amount} min="0" step="0.0001" />
            <button on:click={submitTransaction}>{modalTitle}</button>
        </div>
    </div>
</div>
🔑
