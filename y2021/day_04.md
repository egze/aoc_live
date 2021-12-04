# day4

[![Run in Livebook](https://livebook.dev/badge/v1/blue.svg)](https://livebook.dev/run?url=https%3A%2F%2Fraw.githubusercontent.com%2Fegze%2Faoc_live%2Fmain%2Fy2021%2Fday_04.md)

## Input


```elixir
input = """
7,4,9,5,11,17,23,2,0,14,21,24,10,16,13,6,15,25,12,22,18,20,8,19,3,26,1

22 13 17 11  0
 8  2 23  4 24
21  9 14 16  7
 6 10  3 18  5
 1 12 20 15 19

 3 15  0  2 22
 9 18 13 17  5
19  8  7 25 23
20 11 10 24  4
14 21 16 12  6

14 21 17 24  4
10 16 15  9 19
18  8 23 26 20
22 11 13  6  5
 2  0 12  3  7
"""
```

## Modules

```elixir
defmodule Board do
  defstruct grid: %{}, lookup: %{}

  def new(grid_list) do
    grid =
      for r <- 0..4,
          c <- 0..4,
          into: %{},
          do: {{r, c}, Kernel.get_in(grid_list, [Access.at(r), Access.at(c)])}

    board = %__MODULE__{grid: grid}
    %{board | lookup: Map.new(grid, fn {key, val} -> {val, key} end)}
  end

  def draw_number(board, number) do
    case Map.get(board.lookup, number) do
      {r, c} -> mark(board, {r, c})
      nil -> board
    end
  end

  def winner?(board, number) do
    case Map.get(board.lookup, number) do
      nil -> false
      {r, c} -> check_cell(board, {r, c})
    end
  end

  defp check_cell(board, {r, c} = coords) do
    vertical = for rr <- 0..4, do: {rr, c}
    horizontal = for cc <- 0..4, do: {r, cc}

    Enum.all?(vertical, &(board.grid[&1] == :marked)) or
      Enum.all?(horizontal, &(board.grid[&1] == :marked))
  end

  defp mark(board, coords) do
    %{board | grid: Map.put(board.grid, coords, :marked)}
  end

  def score(board, number) do
    board.grid
    |> Enum.reduce(0, fn
      {_, :marked}, acc -> acc
      {_, value}, acc -> acc + value
    end)
    |> then(fn sum -> sum * number end)
  end
end

defmodule Game do
  defstruct numbers: [], boards: []

  def new do
    %__MODULE__{}
  end

  def add_board(game, board) do
    %{game | boards: [board | game.boards]}
  end

  def draw_number(game, number) do
    %{
      game
      | numbers: [number | game.numbers],
        boards: Enum.map(game.boards, &Board.draw_number(&1, number))
    }
  end

  def winners(game, number) do
    game.boards
    |> Enum.filter(&Board.winner?(&1, number))
  end

  def delete_winners(game, []), do: game
  def delete_winners(game, winners), do: %{game | boards: game.boards -- winners}
end
```

## Parsing

```elixir
{numbers, boards} =
  input
  |> String.split("\n", trim: true)
  |> then(fn [numbers_str | lines] ->
    numbers = numbers_str |> String.split(",") |> Enum.map(&String.to_integer/1)

    boards =
      lines
      |> Enum.map(fn line ->
        line
        |> String.split([" ", "  "], trim: true)
        |> Enum.map(&String.to_integer/1)
      end)
      |> Enum.chunk_every(5)
      |> Enum.map(&Board.new/1)

    {numbers, boards}
  end)

game =
  boards
  |> Enum.reduce(Game.new(), &Game.add_board(&2, &1))
```

## Part1

```elixir
{winner_board, number} =
  numbers
  |> Enum.reduce_while(game, fn number, game_cc ->
    game_acc = Game.draw_number(game_cc, number)
    winner_boards = Game.winners(game_acc, number)

    if Enum.any?(winner_boards), do: {:halt, {hd(winner_boards), number}}, else: {:cont, game_acc}
  end)

Board.score(winner_board, number)
```

## Part 2

```elixir
{winner_board, number} =
  numbers
  |> Enum.reduce_while(game, fn number, game_cc ->
    game_acc = Game.draw_number(game_cc, number)
    winner_boards = Game.winners(game_acc, number)
    game_acc = Game.delete_winners(game_acc, winner_boards)

    case game_acc.boards do
      [] -> {:halt, {hd(winner_boards), number}}
      _ -> {:cont, game_acc}
    end
  end)

Board.score(winner_board, number)
```
