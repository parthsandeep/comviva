# comviva
1 problem-:A barbershop consists of a waiting room with n chairs, and a barber chair for giving haircuts. If there are no customers to be served, the barber goes to sleep. If a customer enters the barbershop and all chairs are occupied, then the customer leaves the shop. If the barber is busy, but chairs are available, then the customer sits in one of the free chairs. If the barber is asleep, the customer wakes up the barber. Write a program to coordinate the interaction between the barber and the customers.

solution-:The solution to this problem includes three semaphores.First is for the customer which counts the number of customers present in the waiting room (customer in the barber chair is not included because he is not waiting). Second, the barber 0 or 1 is used to tell whether the barber is idle or is working, And the third mutex is used to provide the mutual exclusion which is required for the process to execute. In the solution, the customer has the record of the number of customers waiting in the waiting room if the number of customers is equal to the number of chairs in the waiting room then the upcoming customer leaves the barbershop.
code-:Semaphore Customers = 0;
Semaphore Barber = 0;
Mutex Seats = 1;
int FreeSeats = N;

Barber {
	while(true) {
			/* waits for a customer (sleeps). */
			down(Customers);

			/* mutex to protect the number of available seats.*/
			down(Seats);

			/* a chair gets free.*/
			FreeSeats++;
			
			/* bring customer for haircut.*/
			up(Barber);
			
			/* release the mutex on the chair.*/
			up(Seats);
			/* barber is cutting hair.*/
	}
}

Customer {
	while(true) {
			/* protects seats so only 1 customer tries to sit
			in a chair if that's the case.*/
			down(Seats); //This line should not be here.
			if(FreeSeats > 0) {
				
				/* sitting down.*/
				FreeSeats--;
				
				/* notify the barber. */
				up(Customers);
				
				/* release the lock */
				up(Seats);
				
				/* wait in the waiting room if barber is busy. */
				down(Barber);
				// customer is having hair cut
			} else {
				/* release the lock */
				up(Seats);
				// customer leaves
			}
	}
}


second problem-:Typeahead suggestions enable users to search for known and frequently searched terms. As the user types into the search box, it tries to predict the query based on the characters the user has entered and gives a list of suggestions to complete the query. Typeahead suggestions help the user to articulate their search queries better. It’s not about speeding up the search process but rather about guiding the users and lending them a helping hand in constructing their search query. What data structure would be used for the same and how?
solution-:i will use trie data structure and do prefix search after entry of evry character.
code-:
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class Trie {

	public class TrieNode {
		Map<Character, TrieNode> children;
		char c;
		boolean isWord;

		public TrieNode(char c) {
			this.c = c;
			children = new HashMap<>();
		}

		public TrieNode() {
			children = new HashMap<>();
		}

		public void insert(String word) {
			if (word == null || word.isEmpty())
				return;
			char firstChar = word.charAt(0);
			TrieNode child = children.get(firstChar);
			if (child == null) {
				child = new TrieNode(firstChar);
				children.put(firstChar, child);
			}

			if (word.length() > 1)
				child.insert(word.substring(1));
			else
				child.isWord = true;
		}

	}

	TrieNode root;

	public Trie(List<String> words) {
		root = new TrieNode();
		for (String word : words)
			root.insert(word);

	}

	public boolean find(String prefix, boolean exact) {
		TrieNode lastNode = root;
		for (char c : prefix.toCharArray()) {
			lastNode = lastNode.children.get(c);
			if (lastNode == null)
				return false;
		}
		return !exact || lastNode.isWord;
	}

	public boolean find(String prefix) {
		return find(prefix, false);
	}

	public void suggestHelper(TrieNode root, List<String> list, StringBuffer curr) {
		if (root.isWord) {
			list.add(curr.toString());
		}

		if (root.children == null || root.children.isEmpty())
			return;

		for (TrieNode child : root.children.values()) {
			suggestHelper(child, list, curr.append(child.c));
			curr.setLength(curr.length() - 1);
		}
	}

	public List<String> suggest(String prefix) {
		List<String> list = new ArrayList<>();
		TrieNode lastNode = root;
		StringBuffer curr = new StringBuffer();
		for (char c : prefix.toCharArray()) {
			lastNode = lastNode.children.get(c);
			if (lastNode == null)
				return list;
			curr.append(c);
		}
		suggestHelper(lastNode, list, curr);
		return list;
	}


	public static void main(String[] args) {
		List<String> words = List.of("hello", "dog", "hell", "cat", "a", "hel","help","helps","helping");
		Trie trie = new Trie(words);
	
		System.out.println(trie.suggest("hel"));
	}

}
