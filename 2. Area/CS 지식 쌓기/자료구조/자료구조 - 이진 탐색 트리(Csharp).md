```cs
public class BinarySearchTree<T> where T : IComparable<T>
{
	public Node<T> Root { get; private set; }
	
	public bool IsEmpty => Root == null;
	
	
	// 탐색 연산
	public Node<T> Search(T data) => SearchHelper(Root, data);
	private Node<T> SearchHelper(Node<T> node, T data)
	{
		if (node == null) return null;
		
		int compare = data.CompareTo(node.Data);
		
		if (compare == 0) return node;
		return compare < 0 ? SearchHelper(node.Left, data) : SearchHelper(node.Right, data);
	}
	
	// 삽입 연산
	public void Insert(T data)
	{
		Root = InsertHelper(Root, data);
	}
	
	private Node<T> InsertHelper(Node<T> node, T data)
	{
		// 
		if (node == null) return new Node<T>(data);
		
		int compare = data.CompareTo(node.Data);
		
		if (compare < 0)
			node.Left = InsertHelper(node.Left, data);
		else if (compare > 0)
			node.Right = InsertHeper(node.Right, data);
		
		// compare == 0이라면 아무것도 하지 않음 - 이미 있는 값이므로
		
		return node;
	}
	
	// 삭제 연산
	public void Delete(T data)
	{
		Root = DeleteHelper(Root, data);
	}
	
	private Node<T> DeleteHelper(Node<T> node, T data)
	{
		if (node == null) return null;
		
		int compare = data.CompareTo(node.Data);
		
		if (compare < 0 )
		{
			node.Left = DeleteHelper(node.Left, data);
		}
		else if (compare > 0)
		{
			node.Right = DeleteHelper(node.Right, data);
		}
		else 
		{
			// 차수 0 
			if (node.Left == null && node.Right == null) 
				return null;
		
			// 차수 1
			if (node.Left == null) return node.Right;
			if (node.right == null) return node.Left; 
			
			// 차수 2 : 오른쪽 서브트리의 최솟값으로 값을 대체하고 후속자를 삭제
			Node<T> successor = FindMin(node.Right);
			node.Data = successor.Data; // 여기서는 값을 바꿔치기만 함
			node.Right = DeleteHelper(node.Right, successor.Data); // 노드 제거는 재귀 호출로 처리
		}
		
		return node;
	}
	
	private Node<T> FindMin(Node<T> node)
	{
		while (node.Left != null)
			node = node.Left;
			
		return node;
	}
}
```